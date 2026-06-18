---
category: reference
date: 2026-06-18
publish: true
tags:
  - server
  - jellyfin
  - media
  - docker
  - vm
---
# coytishost — Jellyfin Media Stack

I wasn't going to do Jellyfin because I don't watch TV or movies anywhere near enough to warrant it and homelab people annoy me, and never shut up about it.

This got made because I wanted to watch ted lasso and got carried away, the 8tb HDD was bought for this, I spent $350 so I didn't have to spent $20ish on Apple TV

Eat shit Tim Cook

Full self-hosted media stack running in Docker on an Ubuntu 24.04 LTS VM. Jellyfin handles playback; Sonarr/Radarr automate acquisition; qBittorrent runs behind a WireGuard VPN.

---

## VM Specs (VM 105)

| | |
|---|---|
| **OS** | Ubuntu 24.04 LTS |
| **Resources** | 4GB RAM, 4 cores, 32GB OS disk (NVMe) |
| **Media disk** | 8TB virtual disk (`media8tb` pool) — mounted at `/mnt/media`, ~514GB used as of June 2026 |
| **Docker** | Compose-based, files at `/opt/media/docker-compose.yml` |
| **SSH** | Key auth |

---

## Stack

| Container | Port | Purpose |
|-----------|------|---------|
| Jellyfin | 8096 | Media server — [watch.coytis.me](https://watch.coytis.me) |
| Jellyseerr | 5055 | Request portal — [request.coytis.me](https://request.coytis.me) |
| Sonarr | 8989 | TV show automation |
| Radarr | 7878 | Movie automation |
| Prowlarr | 9696 | Indexer manager (synced to Sonarr + Radarr) |
| SABnzbd | 8085 | Usenet downloader |
| Gluetun | — | WireGuard VPN container (Mullvad) |
| qBittorrent | 8080 | Torrent client, routed through Gluetun |

qBittorrent shares Gluetun's network namespace — all torrent traffic exits through the VPN. In Sonarr/Radarr, the download client hostname is `gluetun:8080`, not `qbittorrent`.

---

## Directory Structure

```
/opt/media/          ← Docker Compose + per-container config dirs
/mnt/media/
  tv/                ← TV shows (Sonarr target)
  movies/            ← Movies (Radarr target)
  downloads/         ← SABnzbd/qBittorrent drop zone
```

---

## Useful Commands

```bash
cd /opt/media

docker compose ps              # container status
docker compose logs -f         # live logs (all)
docker compose logs -f jellyfin  # live logs (one container)
docker compose restart         # restart all
docker compose pull && docker compose up -d  # update images
```

---

## Gotchas

- **SABnzbd download paths** — must point to `/downloads/incomplete` (temp) and `/downloads` (complete) on first run, or it fills the 32GB OS disk
- **qBittorrent hostname** — use `gluetun:8080` in Sonarr/Radarr, not `qbittorrent:8080`
- **OS disk** — ~35% used after fixing the SABnzbd issue; Docker images live here, keep an eye on it
- **Port 80** — currently blocked by another host on the LAN. Once resolved: open 80 on UFW + router and Caddy picks it up automatically
