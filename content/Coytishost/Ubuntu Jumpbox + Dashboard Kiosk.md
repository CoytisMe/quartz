---
category: tech
date: 2026-09-25
publish: true
tags:
  - dashboard
  - kiosk
  - xrdp
  - ubuntu
  - homelab
---
# Ubuntu Jumpbox + Dashboard Kiosk

**STATUS: PLANNED, untested**

Goal: one of the spare i5-7500 desktops, running Ubuntu, sitting in the server VLAN. It does two jobs at once:

1. **Dashboard kiosk.** Shows [[log - homepage dashboard setup|Homepage]] (`http://192.168.1.95:3000`) full-screen on a monitor plugged into it.
2. **RDP jumpbox.** A same-subnet desktop to remote into from my PC, for reaching temp VMs by name, Sonarr and other web UIs.

Replaces the [[Pi Zero Dashboard Kiosk]], which I never kept running because of the black flash between minutely updates.

## Why the black screen goes away
The Pi Zero couldn't run a real browser, so it showed a Playwright screenshot through `fbi`. The PNG was re-fetched and redrawn every 60s, which caused the flash. The i5 can run a real browser pointed straight at Homepage. Homepage updates its widgets live in the browser, so there's no reload and no flash. The Playwright screenshotter on the LXC isn't needed for this screen.

## Design: two users, two separate sessions
| User | Session | Role |
|---|---|---|
| `kiosk` | Logs in automatically on the physical console (tty1) | Browser full-screen on the monitor, nothing else |
| `coytis` | Logs in over RDP through xrdp | Own XFCE desktop. The monitor never shows it and it doesn't touch the kiosk |

- xrdp starts a new desktop session for each login rather than mirroring the screen, so remoting in never disturbs the dashboard.
- **Don't use the same user for both.** GNOME in particular has session/dbus conflicts when one user is logged in locally and over RDP at the same time.
- **Why XFCE and not native GNOME remote desktop:** GNOME's "Remote Login" mode also gives a separate session, but it expects a full Ubuntu Desktop install with GDM controlling the screen, which fights with the kiosk for the monitor. xrdp + XFCE stays out of the kiosk's way and runs better over RDP.

## Build (Ubuntu Server 24.04)
```bash
sudo apt install -y cage chromium xrdp xfce4 xfce4-goodies
sudo adduser --disabled-password --gecos "" kiosk
```

### Kiosk side
- Autologin `kiosk` on tty1. Use the same getty override as the Pi, `/etc/systemd/system/getty@tty1.service.d/autologin.conf`:
  ```ini
  [Service]
  ExecStart=
  ExecStart=-/sbin/agetty --autologin kiosk --noclear %I $TERM
  ```
- `~kiosk/.bash_profile`:
  ```bash
  if [ "$(tty)" = "/dev/tty1" ]; then
    exec cage -- chromium --kiosk --noerrdialogs --disable-infobars http://192.168.1.95:3000
  fi
  ```
  `cage` is a minimal Wayland compositor that runs a single app full-screen, so no desktop is needed for the kiosk.
- Add `consoleblank=0` to the kernel command line (`GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub`, then `sudo update-grub`) so the console doesn't blank the screen.
- **Fallback:** if the Chromium snap misbehaves under cage, use `firefox --kiosk http://192.168.1.95:3000` instead.

### Actually installed: Ubuntu Desktop (hostname `coysrvdesk`)
The steps above assume Ubuntu Server. On Desktop, GDM (the login screen) already controls the monitor, so the tty1/cage method doesn't apply. Desktop version:
- **GDM auto-logs in `kiosk`**, never `coytis`. In `/etc/gdm3/custom.conf` under `[daemon]`: `AutomaticLoginEnable=true` and `AutomaticLogin=kiosk`.
- **Browser starts through GNOME autostart:** create `~kiosk/.config/autostart/dashboard.desktop` to launch `firefox --kiosk http://192.168.1.95:3000`.
- **Turn off blanking and the lock screen for `kiosk`:** `gsettings set org.gnome.desktop.session idle-delay 0` and `gsettings set org.gnome.desktop.screensaver lock-enabled false`.
- **Don't** add the `unset DBUS_SESSION_BUS_ADDRESS` / `unset XDG_RUNTIME_DIR` lines to `/etc/xrdp/startwm.sh` that a lot of guides recommend. Without the `dbus-x11` package they make XFCE exit instantly (`dbus-launch ... No such file`). They aren't needed for XFCE anyway.
- **Force XFCE over GNOME:** Ubuntu kept starting a GNOME session over xrdp, which shows black. Fix: `~coytis/.xsessionrc` containing
  ```bash
  export XDG_CURRENT_DESKTOP=XFCE
  export XDG_SESSION_DESKTOP=xfce
  export DESKTOP_SESSION=xfce
  ```
- **Status 2026-09-25:** xrdp + XFCE working as `coytis`. It's slower than Ubuntu's own remote desktop, which streams using the graphics card; xrdp on 24.04 does everything on the CPU. Speed tweaks to try:
  - Turn off the XFCE compositor: Settings > Window Manager Tweaks > Compositor
  - In `mstsc`: Experience tab, set **LAN (10 Mbps or higher)**
  - Bigger network buffers: `tcp_send_buffer_bytes=4194304` under `[Globals]` in `/etc/xrdp/xrdp.ini`, plus `net.core.wmem_max = 8388608` in `/etc/sysctl.d/60-xrdp.conf`, then restart xrdp
  - Turn off smooth scrolling in the browser
  - If it's still too slow: try GNOME's "Remote Login" instead. It might be fine alongside the kiosk since they're different users. Untested.
- **Stale sessions:** xrdp reconnects you to your existing session, so if one is stuck (e.g. a leftover black GNOME one), kill it before trying again.
- **Black screen over RDP** = `coytis` is probably also logged in on the physical screen. Only `kiosk` should ever be logged in locally.

### Remote side
- As `coytis`: `echo xfce4-session > ~/.xsession`
- Connect with `mstsc` from my PC.
- At the xrdp login box, pick session **Xorg**. Xvnc is the older, slower backend, and vnc-any / neutrinordp-any are for proxying to other machines.

## XFCE vs the GNOME I'm used to
- **Looks different:** instead of GNOME's dock and Activities overview, there's a panel with an Applications menu, taskbar and tray, a bit like older Windows. Plainer, and settings are spread across several small apps.
- **Works the same:** same Firefox/Chromium (Sonarr and the other web UIs are identical), terminal, file manager, copy/paste between my PC and the session. VM names resolve because the box is on the same subnet.
- Themes can make it fairly GNOME-like if the look bugs me.

## Alternative: no GUI needed
If all I need is to reach VMs by name, the box also works as an SSH jump host:
- `ssh -J coytis@jumpbox vm-name` for shells
- `ssh -D 1080 coytis@jumpbox` plus a browser profile set to that SOCKS proxy, with remote DNS turned on, for VM web UIs

## Next steps
- [ ] Once working, move to `KB/Coytishost/` and update [[Pi Zero Dashboard Kiosk]] to point here
