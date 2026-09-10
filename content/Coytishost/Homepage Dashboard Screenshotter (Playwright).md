---
category: tech
date: 2026-08-25
publish: true
tags:
  - raspberry-pi
  - dashboard
  - homelab
  - playwright
---

# Homepage Dashboard Screenshotter (Playwright)

Plan C for [[Pi Zero Dashboard Kiosk]] — the Pi Zero (ARMv6) can't run a modern browser engine reliably (Chromium needs NEON it doesn't have, WebKitGTK/surf SIGFPEs on Homepage's React content). Instead of rendering on the Pi, render on the same LXC that already hosts [[log - homepage dashboard setup|Homepage]] (LXC 112, `192.168.1.95`, 2 core/2GB RAM, **5GB disk free** as of 2026-08-25) using Playwright, and have the Pi just display the resulting PNG via `fbi` (framebuffer image viewer — no X, no browser, no GL).

Deploy as two more services in the same `docker-compose.yml` as Homepage/Kuma, so they land on the existing `homepage_default` network automatically and can reach Homepage via its Compose service name — same hairpin-NAT gotcha Kuma already hit and fixed (see [[log - homepage dashboard setup]], bug #1): **must use `http://homepage:3000`, not the LAN IP `192.168.1.95:3000`**, from inside another container on this host.

## 1. Screenshotter service

Local mirror path (matching the existing convention): `C:\Users\Rick\VMHost\homepage\screenshotter\`
Deployed path on LXC: `/opt/homepage/screenshotter/`

**`screenshotter/Dockerfile`**
```dockerfile
FROM mcr.microsoft.com/playwright:v1.48.0-noble
WORKDIR /app
COPY package.json .
RUN npm install
COPY shoot.js .
CMD ["node", "shoot.js"]
```

**`screenshotter/package.json`**
```json
{
  "name": "dashboard-screenshotter",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "playwright": "1.48.0"
  }
}
```

**`screenshotter/shoot.js`**
```js
const { chromium } = require('playwright');
const fs = require('fs');

const URL = process.env.TARGET_URL || 'http://homepage:3000';
const OUT = process.env.OUT_PATH || '/screenshots/dash.png';
const INTERVAL_MS = parseInt(process.env.INTERVAL_MS || '60000', 10);
const WIDTH = parseInt(process.env.WIDTH || '1920', 10);
const HEIGHT = parseInt(process.env.HEIGHT || '1080', 10);

async function shoot(browser) {
  const page = await browser.newPage({ viewport: { width: WIDTH, height: HEIGHT } });
  try {
    await page.goto(URL, { waitUntil: 'networkidle', timeout: 30000 });
    await page.waitForTimeout(1000); // let widgets finish their own data fetches
    const tmp = OUT + '.tmp';
    await page.screenshot({ path: tmp, type: 'png' }); // type must be explicit — screenshot() infers format from the path's extension, and ".tmp" isn't recognized
    fs.renameSync(tmp, OUT); // atomic swap so the Pi never fetches a half-written file
    console.log(new Date().toISOString(), 'screenshot ok');
  } catch (err) {
    console.error(new Date().toISOString(), 'screenshot failed:', err.message);
  } finally {
    await page.close();
  }
}

(async () => {
  const browser = await chromium.launch();
  await shoot(browser);
  setInterval(() => shoot(browser), INTERVAL_MS);
  process.on('SIGTERM', async () => { await browser.close(); process.exit(0); });
})();
```

Runs as a long-lived container (not a per-shot cron job) — Chromium stays warm between shots so each screenshot is fast, and `setInterval` keeps the cadence simple. RAM cost is one persistent headless Chromium instance rather than a repeated cold-start; worth watching `docker stats` after deploy given the 2GB ceiling shared with Homepage + Kuma.

## 2. Static file server (serves the PNG to the Pi)

```yaml
  screenshot-server:
    image: nginx:alpine
    container_name: dashboard-screenshot-server
    restart: unless-stopped
    volumes:
      - screenshots:/usr/share/nginx/html:ro
    ports:
      - "8080:80"
```

## 3. Add both to the existing `docker-compose.yml`

```yaml
  screenshotter:
    build: ./screenshotter
    container_name: dashboard-screenshotter
    restart: unless-stopped
    environment:
      - TARGET_URL=http://homepage:3000
      - OUT_PATH=/screenshots/dash.png
      - INTERVAL_MS=60000
    volumes:
      - screenshots:/screenshots
    depends_on:
      - homepage

  screenshot-server:
    image: nginx:alpine
    container_name: dashboard-screenshot-server
    restart: unless-stopped
    volumes:
      - screenshots:/usr/share/nginx/html:ro
    ports:
      - "8080:80"

volumes:
  screenshots:
```

Deploy: `docker compose up -d --build` from `/opt/homepage/`.

## 4. UFW — open 8080, same LAN-only pattern as 3000/3001

```bash
sudo ufw allow from 192.168.1.0/24 to any port 8080 proto tcp
sudo ufw allow from 192.168.2.0/24 to any port 8080 proto tcp
```

## 5. Pi side — fetch + display loop (final, working)

**STATUS: WORKING**, confirmed live 2026-08-29. The version below is the corrected final one — see [[Pi Zero Dashboard Kiosk]] for the full debugging story (three stacked bugs: systemd `TTYPath` couldn't do VT ioctls, `-t` timeout didn't fix a mystery instant-exit, and the actual root cause — a backgrounded `&` process in a `#!/bin/sh` script gets its stdin silently redirected to `/dev/null` by POSIX shells, which broke fbi's VT ioctls in every single attempt regardless of session type).

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

Key details that mattered: `-d /dev/fb0` forces plain framebuffer mode (skips fbi's DRM auto-detect VT-switch path); `< /dev/tty1` explicitly gives fbi a real tty on stdin (without this, VT ioctls fail no matter what); `-t 65` is a safety-net timeout slightly longer than the loop's own 60s cadence. Launched via genuine tty1 autologin, not a bare systemd service — a `dash-display.service` with `TTYPath=/dev/tty1` was tried first and didn't work (that's dead-end #1 in the debugging log). **This replaces the entire X/browser stack from [[Pi Zero Dashboard Kiosk]]** — no X server, no `.xinitrc`, no `startx`, no browser at all; `fbi` runs directly on the console framebuffer.

## Disk footprint (confirmed)
`mcr.microsoft.com/playwright` base image landed at ~3.1GB actual usage on the LXC (72% of 7.8GB, 2.2GB free remaining) — a bit more than the earlier ~1.5–2GB estimate, but still comfortable headroom alongside Homepage + Kuma.

## Next steps
- [x] Create `screenshotter/` folder, deploy to LXC at `/opt/homepage/screenshotter/`
- [x] Add both services to `/opt/homepage/docker-compose.yml`, `docker compose up -d --build`
- [x] Confirmed DNS resolution + screenshot working (`docker logs dashboard-screenshotter` → "screenshot ok")
- [x] Confirmed `http://192.168.1.95:8080/dash.png` serves a real 200 OK PNG from the LAN
- [x] Checked `df -h` on LXC post-build — 2.2GB free, fine
- [x] Pi's `.bash_profile` reworked to drop X entirely and run the `fbi` fetch-loop instead — **live and working**
- [x] Consider a systemd service instead of autologin for the Pi-side loop now that the stdin bug is understood (`TTYPath` + `< /dev/tty1` would likely work now) — not urgent, current setup works fine ✅ 2026-09-02
- [x] Fixed a bug in `shoot.js` along the way: `page.screenshot({ path })` infers format from the file extension, and the `.tmp` temp-file suffix broke that — needed explicit `type: 'png'` (already corrected above) ✅ 2026-09-02
