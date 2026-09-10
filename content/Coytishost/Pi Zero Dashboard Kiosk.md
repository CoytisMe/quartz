---
category: tech
date: 2026-08-24
publish: true
tags:
  - raspberry-pi
  - dashboard
  - kiosk
---
# Pi Zero Dashboard Kiosk

**STATUS: WORKING**, as of 2026-08-29 — see [[Homepage Dashboard Screenshotter (Playwright)]] for the final architecture. Browser-in-kiosk approach (below) was abandoned as a dead end; the working solution renders on the LXC via Playwright and displays a static PNG on the Pi via `fbi`, no browser/X involved at all.

Goal: $30 Raspberry Pi Zero + spare 1080p (LG) monitor as an always-on display for the [[log - homepage dashboard setup|Homepage]] dashboard at `http://192.168.1.95:3000`.

## Static IP: 
192.168.2.97
## Confirmed hardware/OS
- **Original Pi Zero/Zero W (ARMv6, no NEON)** — confirmed via chromium's NEON error and later `armv6l` in kernel banner. Not a Zero 2 W.
- Running **Raspberry Pi OS Lite** (Bookworm-based, `armv6l`), set up via `rpi-imager` with cloud-init (`ds=nocloud`).
- Hostname `coytisboard`, user `coytis`, static-ish on `192.168.1.x` / `192.168.2.x` LAN.
- Monitor: LG FULL HD, confirmed via EDID in Xorg log — X drives it fine at 1920x1080.

## Dead ends (all confirmed, don't retry blindly)

1. **Chromium (`chromium`/`chromium-browser`)** — dead on arrival. Debian/Raspberry Pi OS Bookworm's stock `chromium` package requires NEON (ARMv7+). This is a hard CPU incompatibility, not fixable by reflashing or reinstalling — RPi OS now ships the same NEON-requiring Debian build (no longer maintains their own ARMv6-compatible fork).
2. **Epiphany (GNOME Web)** — `--application-mode <URL>` doesn't work as expected; it wants a pre-installed `.desktop` web-app file, not a raw URL (`Invalid desktop file passed to --application-mode`). Switched to plain `epiphany <URL>` + xdotool F11 — got past that, but then hit:
3. **Epiphany GL/EGL failure** — `Unable to create a GL context` / `No EGL configuration available`. X's own glamor/DRI2 GL works fine (confirmed in Xorg log — VC4 V3D acceleration loads correctly), but WebKitGTK's own separate EGL context creation fails. Tried `WEBKIT_DISABLE_COMPOSITING_MODE=1` — screen stayed black (unclear if this alone would've fixed Epiphany specifically; moved to surf before fully isolating).
4. **surf (suckless browser, same WebKitGTK backend)** — got furthest here. X launches perfectly (verified full clean Xorg log, HDMI-1 connected, EDID read correctly, VC4 GL accelerated). Surf itself launches, runs ~28s, then **crashes with "Floating point exception" (SIGFPE)** — confirmed via `xinit` log, not a kernel OOM (checked `dmesg`/`free -h`, both clean) and not an X-level crash. Tried `JSC_useJIT=0` to force JavaScriptCore into pure-interpreter mode (ruling out a JIT codegen bug) — **same SIGFPE**, so the crash is not in the JIT, it's deeper in WebKit (likely a genuine ARMv6 float-handling bug provoked by Homepage's React/JS rendering — chart widgets, icon libraries etc. are the usual suspects for this class of bug).

Abandoned here — never ran the planned `info.cern.ch` isolation test. Pivoted straight to Plan C instead, which turned out to be the right call.

## Key debugging gotchas learned (useful beyond this project too)
- `xorg` is a metapackage name, not a command.
- X refuses to start over an SSH session even with `Xwrapper.config allowed_users=anybody` set — it also needs genuine console/VT access (`/dev/tty0`), which only a real tty1 login session gets via systemd-logind. **Can't test kiosk X sessions over SSH directly** — must either physically watch the monitor after a real console login, or force a fresh tty1 login remotely via `sudo systemctl restart getty@tty1.service` (this re-triggers `.bash_profile` → `startx` without a full reboot).
- Redirecting `startx` output to a file (`> xstart.log 2>&1`) can appear truncated when cat'd immediately — stdio fully-buffers when not attached to a tty, so partial reads mid-session are misleading. Xorg's own log at `~/.local/share/xorg/Xorg.0.log` flushes properly and is more reliable for mid-session checks. Once the session has *fully exited*, the redirected file is reliable too (that's how the "Floating point exception" line was actually found).

## Current `.bash_profile` (working, don't need to touch again)
```bash
if [ -z "$DISPLAY" ] && [ "$(tty)" = "/dev/tty1" ]; then
  startx -- -nocursor
fi
```

## Current `.xinitrc` (X launch works; browser client crashes)
```bash
#!/bin/sh
xset s off
xset s noblank
xset -dpms

export WEBKIT_DISABLE_COMPOSITING_MODE=1
export JSC_useJIT=0

(sleep 15 && while true; do sleep 60; xdotool key --clearmodifiers F5; done) &

surf -F http://192.168.1.95:3000
```
Note: the F5-refresh loop is actually unnecessary — Homepage auto-updates its widgets client-side already. Can be dropped once something renders.

Also set: `hdmi_force_hotplug=1` in `/boot/firmware/config.txt`, `allowed_users=anybody` in `/etc/X11/Xwrapper.config`, autologin on tty1 via `raspi-config nonint do_boot_behaviour B2`.

## Plan C — the winning approach (now live)
Render on the LXC instead of the Pi Zero — it's fundamentally too weak/incompatible for a modern JS-heavy dashboard (Chromium needs NEON it doesn't have; WebKitGTK crashes on Homepage's content). Full build-out, script, and Docker Compose config: [[Homepage Dashboard Screenshotter (Playwright)]]. Pi-side deployment gotchas (the fun part) are below.

### Getting `fbi` actually working on the Pi — three separate bugs, stacked
Once the LXC side (Playwright + nginx) was up and serving a real PNG, the Pi side still took three rounds of debugging before `fbi` would stay on screen:

1. **fbi under a bare systemd service (`TTYPath=/dev/tty1`) couldn't do VT ioctls** — `ioctl VT_GETSTATE/VT_GETMODE: Inappropriate ioctl for device`. Assumed this meant it needed a genuine console/logind session (same as X did earlier) — reasonable theory, but wrong, see #3.
2. **fbi would flash the image for under a second then exit** — no crash, no error, just silently gone. Assumed it was fbi's keypress-wait logic misfiring on a broken stdin; adding `-t 65` (auto-timeout instead of waiting for a key) didn't fix it — ruled that theory out.
3. **Actual root cause**: the script backgrounds fbi with a plain `&` inside a `#!/bin/sh` loop. **POSIX shells (dash, which is Debian's `/bin/sh`) redirect a backgrounded job's stdin to `/dev/null` by default**, regardless of the script's own controlling terminal. So fbi's stdin was never `/dev/tty1` in *any* attempt — systemd service, SSH, or genuine tty1 autologin — it was always `/dev/null`, which explains the identical `VT_GETSTATE` failure no matter how the session was launched. Fix: explicitly redirect fbi's stdin — `fbi ... < /dev/tty1 > /tmp/fbi.log 2>&1 &`. That one flag fixed it after the session-type red herring wasted real time.

**Worth remembering generally**: any time a backgrounded (`&`) process inside a shell script needs a real tty (VT ioctls, terminal-aware CLI tools, etc.), redirect its stdin explicitly — don't assume it inherits the script's own terminal.

### Final working Pi-side setup
`~/.bash_profile`:
```bash
if [ -z "$DISPLAY" ] && [ "$(tty)" = "/dev/tty1" ]; then
  exec /home/coytis/dash-display.sh
fi
```

`/home/coytis/dash-display.sh`:
```bash
#!/bin/sh
while true; do
  curl -fsS -o /tmp/dash.png.tmp http://192.168.1.95:8080/dash.png && mv /tmp/dash.png.tmp /tmp/dash.png
  pkill fbi 2>/dev/null
  fbi -d /dev/fb0 -a --noverbose -t 65 /tmp/dash.png < /dev/tty1 > /tmp/fbi.log 2>&1 &
  sleep 60
done
```

Launched via genuine tty1 autologin (`getty@tty1.service` + `/etc/systemd/system/getty@tty1.service.d/autologin.conf`, same override used for the abandoned X approach) — **not** a standalone systemd service with `TTYPath`, which is what caused bug #1 above (a `dash-display.service` unit was tried and scrapped in favor of this).

`-d /dev/fb0` forces plain framebuffer mode (skips fbi's DRM auto-detect path entirely). No X, no `.xinitrc`, no browser — this fully replaces the entire dead-end setup above.

## Smoother image transition (queued, untested)
Current script kills and relaunches `fbi` every cycle — the visible "flash" is mostly the process restart (reopening the framebuffer device, reloading the font) rather than the image change itself. `fbi` has no crossfade/blend capability at all (it's a raw framebuffer writer, no compositing), so a true smooth dissolve isn't achievable on this hardware — but the restart-flash itself is fixable: keep one `fbi` process alive permanently and use its remote-control FIFO (`--comm`) to trigger a redraw instead of restarting the process each time.

```bash
#!/bin/sh
FIFO=/tmp/fbi-ctrl
[ -p "$FIFO" ] || mkfifo "$FIFO"

fbi -u -a --noverbose -d /dev/fb0 --comm "$FIFO" /tmp/dash.png < /dev/tty1 > /tmp/fbi.log 2>&1 &

while true; do
  curl -fsS -o /tmp/dash.png.tmp http://192.168.1.95:8080/dash.png && mv /tmp/dash.png.tmp /tmp/dash.png
  echo > "$FIFO"
  sleep 60
done
```

`fbi` now launches once outside the loop with `-u` (don't cache, always re-read from disk) and `--comm "$FIFO"`; the loop only fetches the image and pokes the FIFO with a blank line to trigger a redraw — no more `pkill`/relaunch. **Not yet confirmed**: whether a bare `echo >` is actually the right trigger command for fbi's comm protocol — check `man fbi` before relying on this, might need a specific keyword instead of a blank line.

## Rotating for a vertical/portrait monitor
`fbi` has no built-in rotation flag — rotate the image file itself before displaying it. Simplest approach, works regardless of what the LXC-side Playwright script captures:

```bash
sudo apt install -y imagemagick
```

Add a rotate step in `dash-display.sh` right after the `curl`/`mv`, before `fbi` runs:

```bash
convert /tmp/dash.png -rotate 90 /tmp/dash.png
```

Use `-rotate 90` or `-rotate 270` depending on which way the monitor is physically turned — try one, and if the dashboard reads upside-down or sideways, switch to the other.

**Optional refinement, not required to just get it working:** Homepage is likely responsive, so instead of capturing a landscape (1920x1080) screenshot and then rotating it — which just tilts the landscape layout sideways, small text and all — you could swap `WIDTH`/`HEIGHT` in the screenshotter's `docker-compose.yml` environment (e.g. `WIDTH=1080`, `HEIGHT=1920`) so Homepage's own CSS reflows into a genuine portrait layout, then still apply the same `-rotate 90` step on the Pi to match the physical panel's native landscape scan-out. Untested — depends on how well Homepage's layout actually reflows at a narrow viewport.

## Next steps
- [x] Build Plan C (screenshot source + `fbi` loop on the Pi) — **done and confirmed working 2026-08-29**
- [x] Consider a cleaner systemd-based launch for the Pi side now that the stdin bug is understood (the `TTYPath` approach would work fine now with `< /dev/tty1` added) — not urgent, current autologin setup works ✅ 2026-09-02
- [x] Try the `--comm` FIFO approach above to smooth out the transition flash ✅ 2026-09-02
- [x] If going vertical: install imagemagick + add the rotate step, decide 90 vs 270 by testing ✅ 2026-09-02
- [x] Longer-term: notification wiring for Uptime Kuma is still outstanding per [[log - homepage dashboard setup]] — unrelated to this note but flagged there ✅ 2026-09-02
