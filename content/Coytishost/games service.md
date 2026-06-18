---
category: reference
date: 2026-06-18
publish: true
tags:
  - server
  - gaming
  - minecraft
  - factorio
  - satisfactory
  - lxc
---

# The servers

Three game servers running as Proxmox LXC containers on coytishost. All auto-start on host boot.

---

## Minecraft (LXC 101)

Vanilla Minecraft survival server.

1 person uses it, and it's not me

| | |
|---|---|
| **OS** | Debian 12 (Bookworm) |
| **Resources** | 6GB RAM, 4 cores, 20GB NVMe |
| **Java** | Eclipse Temurin 25 JRE (Adoptium) |
| **Version** | Vanilla 1.21.11 |
| **World** | QUARANTINIA |
| **Port** | TCP 25565 |
| **Whitelist** | Enabled, online-mode on |
| **Service** | systemd (`minecraft.service`), enabled on boot |
| **JVM args** | `-Xmx6144M -Xms2048M` |

```bash
systemctl status minecraft
journalctl -u minecraft -f
systemctl restart minecraft
```

---

## Factorio (LXC 102)

Factorio dedicated server managed by OFSM (Open Factorio Server Manager) running in Docker

This has my new Space Age save on it, the one I can't play unless I'm streaming, therefore it lays dormant as my omnipresent reminder that i'm somehow too mental ill to stream

| | |
|---|---|
| **OS** | Debian 12 (Bookworm) |
| **Resources** | 4GB RAM, 2 cores, 20GB NVMe |
| **Docker** | CE 29.5.2 |
| **Manager** | OFSM (`ofsm/ofsm:latest`) via Docker Compose |
| **Game version** | 2.0.76 (pinned to match client) |
| **DLC** | Space Age (built into binary) |
| **Port** | UDP 34197 |
| **Web UI** | Port 8080 |
| **Mods** | even-distribution, factoryplanner, flib |

```bash
# SSH into LXC, then:
docker ps
docker logs factorio -f
cd /opt/factorio && docker compose restart
```

> To update: change `FACTORIO_VERSION` in `/opt/factorio/docker-compose.yml` to match Steam client version, then `docker compose pull && docker compose up -d`.

---

## Satisfactory (LXC 103)

Satisfactory dedicated server via SteamCMD, running as the `steam` user (required — server refuses to run as root).

Twas for Bronson and I, a save we started right before Diablo 4 released something and then he got a job. I keep the server alive like a civil war era woman keeping her front porch lit in vain hope of a knock on the door freeing her from the moniker of 'widow'.

| | |
|---|---|
| **OS** | Debian 12 (Bookworm) |
| **Resources** | 12GB RAM, 4 cores, 40GB NVMe |
| **Install** | SteamCMD, App ID 1690800, anonymous login |
| **Install path** | `/opt/satisfactory/` |
| **Service** | systemd (`satisfactory.service`), enabled on boot |
| **Ports** | UDP 7777 (game), UDP 15000 (beacon), UDP 15777 (query) |
| **Saves** | `/home/steam/.config/Epic/FactoryGame/Saved/SaveGames/server/` |

```bash
systemctl status satisfactory
journalctl -u satisfactory -f
systemctl restart satisfactory

# Update
systemctl stop satisfactory
su - steam -c "/opt/steamcmd/steamcmd.sh +force_install_dir /opt/satisfactory +login anonymous +app_update 1690800 +quit"
systemctl start satisfactory
```

> On first connection, claim the admin password via the in-game server browser before anyone else connects.
