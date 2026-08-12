---
category: reference
date: 2026-08-08
publish: true
tags:
  - server
  - jellyfin
  - homelab
  - troubleshooting
---
## Jellyfin Logs & Transcode Triage

For when someone says "it's not working" and you need to find out if that's true.

Jellyfin lives on the media server (`.96`) in Docker. Logs are on the host disk, so you don't need to go inside the container to read them.

---
## Where the logs are

```bash
ssh rick@192.168.1.96
ls -lt /opt/media/jellyfin/config/log/
```

Inside the container that same folder is `/config/log`. Three kinds of file:

| File | What it is |
|---|---|
| `log_YYYYMMDD.log` | The main server log. One per day. Start here. |
| `FFmpeg.Transcode-<date>_<mediasourceid>_<session>.log` | One per transcode. The full ffmpeg command + everything it printed. |
| `FFmpeg.DirectStream-...log` | Same but for remux (repackaging, no re-encode). Cheap. |

The long hex in the ffmpeg filename is the **media source ID**, not the user or device. Same episode = same ID every time, which is handy for grouping attempts at one file.

Live tail:
```bash
docker logs -f jellyfin
tail -f /opt/media/jellyfin/config/log/log_$(date +%Y%m%d).log
```

---
## Reading a playback session

A normal session looks like this, in order:

```
MediaInfoHelper: User policy for "Someone". EnablePlaybackRemuxing: True ...
TranscodeManager: "/usr/lib/jellyfin-ffmpeg/ffmpeg" -i file:"/media/tv/..." ...
SessionManager: Playback start reported by app "Jellyfin Web" ...
SessionManager: Playback stopped reported by app ... Stopped at "253741" ms
TranscodeManager: FFmpeg exited with code 0
```

Useful greps:

```bash
# everything one user did today
grep -i 'username' /opt/media/jellyfin/config/log/log_$(date +%Y%m%d).log

# real problems only
grep -E '\[ERR\]|\[WRN\]' /opt/media/jellyfin/config/log/log_$(date +%Y%m%d).log

# what actually got played and how far
grep 'Playback stopped' /opt/media/jellyfin/config/log/log_$(date +%Y%m%d).log
```

**The single most useful tell:** a transcode gets spawned but there is **no** "Playback start reported" after it. That means the server did its job and the client never started. Don't go hunting for file corruption — the problem is at the client end or it's a bad resume position (see below).

---
## Was it transcoding, and why?

Look at the ffmpeg command in the log.

- `-codec:v:0 copy` → **direct stream / remux.** Basically free.
- `-codec:v:0 libx264` → **software transcode.** This is the expensive one.
- `-codec:a:0 copy` → audio passed through untouched.
- `-ss 00:04:12.336` → the user **resumed** from that point rather than starting at 0.
- `-maxrate` → the bitrate ceiling the client asked for. Low value = client on a slow connection or someone set a quality cap.

The `speed=Nx` lines in the ffmpeg log are the health check:

- **speed above ~2x** — fine, encoder is ahead of the viewer.
- **speed near or below 1x** — the encoder cannot keep up with playback. This is what buffering actually looks like from the server side.

---
## Know your ceiling

This matters more as usage grows, so read it once:

- The media VM has **4 vCPUs**.
- There is **no GPU** in it — `/dev/dri` doesn't exist and `HardwareAccelerationType` is `none`.
- So **every transcode is software x264 on 4 cores.**

The old DVD-rip stuff (480p XviD) encodes at 20x+ and costs nothing. Modern 1080p, and especially anything 4K or HEVC, is a completely different story — expect roughly **one or two 1080p transcodes at once** before things start stuttering for everybody.

Ways to buy headroom, cheapest first:

1. **Stop the transcodes happening.** Most are triggered by the client, not the file — someone's app is set to a quality cap instead of "Auto", or is on a browser that won't take the container. Fixing one client's settings beats any server tuning.
2. **Give the VM more cores.** It's on an i9-9900K, 4 is stingy.
3. **Pass through the 1080Ti** and set hardware acceleration to NVENC. Biggest win by far, but the GPU is spoken for — see [[GPU Passthrough to Windows VM]].

Check what's running right now:
```bash
docker exec jellyfin ps aux | grep ffmpeg | grep -v grep    # active transcodes
docker stats --no-stream jellyfin                            # CPU being used
```

Note `EnableThrottling` is currently **false**, so ffmpeg races ahead and encodes the whole file as fast as it can even if the viewer stops after two minutes. Wasted CPU and — worse on a single spinning disk — wasted I/O. Worth turning on if the server starts feeling busy. Related: the SABnzbd I/O starvation notes in the Jellyseerr overload log.

---
## "X can't watch episode Y"

Learned this the hard way in Aug 2026, after scanning an entire episode for corruption that was never there.

**Check the watch state before you touch anything else.** There's no `sqlite3` on the host, but Python has it:

```bash
python3 -c "
import sqlite3
c=sqlite3.connect('/opt/media/jellyfin/config/data/jellyfin.db'); c.row_factory=sqlite3.Row
for r in c.execute('''select u.Username, ud.PlaybackPositionTicks p, ud.Played, ud.PlayCount
 from UserData ud join BaseItems b on b.Id=ud.ItemId join Users u on u.Id=ud.UserId
 where b.Name like \"%EPISODE NAME%\"'''):
    print(r['Username'], round((r['p'] or 0)/10000000,1),'s  played=',r['Played'],' n=',r['PlayCount'])
"
```

A **non-zero position with `played=0`** is a stranded resume point. Every time they press play it seeks into that spot and the client never starts — which is exactly the "transcode spawned, no playback start" pattern above.

**Fix: tick and untick.** Mark the item watched, then unwatched, on that account. Plain "Play from beginning" does not reliably clear it.

Also — **"but I can watch it fine" is not evidence.** Check your own position too. If it says 2 seconds, you never reached the failure point either. Play through the actual spot before blaming anyone's internet.

---
## Red herrings

These show up constantly and mean nothing:

- `[mp3float] Header missing` — artifact of seeking into an MP3 stream. Present in files that play perfectly.
- `Video uses a non-standard and wasteful way to store B-frames ('packed B-frames')` — normal for old XviD rips. Cosmetic.
- `Library folder "/config/data/playlists" is inaccessible or empty, skipping` — harmless.
- Caddy `aborting with incomplete response ... okhttp/4.12.0` — that's the Android TV app cancelling a stream it was done with. Normal.
- `FFmpeg exited with code 0` after `Stopping ffmpeg process with q command` — that's a clean shutdown when someone stops watching, not a crash.

Actually worth caring about: `[ERR]` lines, exit codes **other than 0**, `speed=` below 1x, and transcodes with no playback start after them.

---
## Proving it yourself

To test a file without involving anyone's client, use the ffmpeg that ships inside the container:

```bash
# does the file decode cleanly end to end?
docker exec jellyfin /usr/lib/jellyfin-ffmpeg/ffmpeg -v error \
  -i "/media/tv/Show/file.avi" -f null -
```

Silence means the file is fine. Remember the host's `/mnt/media/tv` is `/media/tv` inside the container.

If you're re-running one of Jellyfin's own transcode commands to reproduce something: **don't add `-t`**. Jellyfin uses `-copyts`, and `-t` gets measured against the original timestamps, so you get an empty output that looks like a dramatic bug and isn't. Use `-frames:v 600` to cap it instead.

---
Related: [[The Jellyfin Stack]] · [[Linux commands]] · [[coytishost & Proxmox]]
