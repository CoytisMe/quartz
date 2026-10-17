---
category: tech
date: 2026-09-27
publish: true
source:
tags:
  - linux
  - kiosk
  - dashboard
  - gnome
  - homelab
---
# Kiosk Monitor Off at Night

Scripts to have the dashboard monitor on the [[Ubuntu Jumpbox + Dashboard Kiosk]] turn off at night.

Turn display off
Saved as monitorsoff
```bash
busctl --user set-property org.gnome.Mutter.DisplayConfig /org/gnome/Mutter/DisplayConfig org.gnome.Mutter.DisplayConfig PowerSaveMode i 1
```
Turn display on
Saved as monitorson
```bash
busctl --user set-property org.gnome.Mutter.DisplayConfig /org/gnome/Mutter/DisplayConfig org.gnome.Mutter.DisplayConfig PowerSaveMode i 0
```
Has to be ran on Kiosk Account

### Crontab
crontab -e
```bash
0 23 * * * /home/kiosk/scripts/monitorsoff.sh
0 7 * * * /home/kiosk/scripts/monitorson.sh
```

#### Why the crontab didn't fire
`busctl --user` talks to the kiosk user's session D-Bus. A terminal or SSH login sets up the variables that point at it (`XDG_RUNTIME_DIR`, `DBUS_SESSION_BUS_ADDRESS`). Cron runs with an almost empty environment, so `busctl` can't find the bus and fails without saying anything. The fix is to set both variables at the top of each script:

```bash
#!/bin/bash
export XDG_RUNTIME_DIR=/run/user/$(id -u)
export DBUS_SESSION_BUS_ADDRESS=unix:path=$XDG_RUNTIME_DIR/bus
busctl --user set-property org.gnome.Mutter.DisplayConfig /org/gnome/Mutter/DisplayConfig org.gnome.Mutter.DisplayConfig PowerSaveMode i 1
```
(`i 0` in `monitorson.sh`.)

Also check:
- The crontab is the **kiosk** user's (`crontab -l` as kiosk), not root's or coytis's.
- The scripts are executable: `chmod +x /home/kiosk/scripts/monitor*.sh`.
- The path in the crontab (`/home/kiosk/scripts/monitorsoff.sh`) is the real file. The SSH command calls `monitorsoff` with no `.sh`, so it may be running a different copy.
- To see what cron is doing, send its output to a log: `0 23 * * * /home/kiosk/scripts/monitorsoff.sh >> /home/kiosk/scripts/cron.log 2>&1`. Also check `grep CRON /var/log/syslog`.
- To test quickly without waiting for 11pm, set a cron entry 2 minutes ahead, or run the script with an empty environment the way cron does: `env -i /home/kiosk/scripts/monitorsoff.sh`.

**Scripts to do it through SSH**
Set up on the X1
```bash
#!/bin/bash
ssh kiosk@192.168.1.67 "monitorsoff"
```
You can guess how to make the monitorson one.

Related: [[Linux commands]], [[GNOME Super Key Fix]]

## Followup
- [x] `192.168.1.67` added to [[Network Reference]] as coysrvdesk ✅ 2026-09-28
- [x] Scripts saved as `monitorsoff`/`monitorson` on coysrvdesk, fired by kiosk's crontab ✅ 2026-09-28
- [x] Add the two `export` lines to both scripts and confirm cron actually turns the screen off at 23:00. The fix hasn't been tested yet. ✅ 2026-10-17
