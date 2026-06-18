---
category: reference
date: 2026-06-18
publish: true
tags:
  - server
  - proxmox
  - hardware
  - homelab
---

# coytishost — Host & Proxmox

Repurposed gaming/streaming PC running Proxmox VE as a bare-metal homelab host.

---

## Hardware
This PC was given to me by my boss after my first year working at my company 5 years ago. The 8tb drive and the 1080ti added later,

We had two of them, identical specs, both from one client We were trying to sell because they had swapped to a laptop.

That client was a conveyancer, a conveyancer had two 9900k machine with 64gb RAM

| Component | Spec |
|-----------|------|
| CPU | Intel Core i9-9900KF — 8c/16t, 3.6GHz base / 5.0GHz boost, LGA1151 |
| RAM | 64GB DDR4-2666 (4x 16GB A-DATA, dual channel) |
| Motherboard | MSI MPG Z390M GAMING EDGE AC (mATX, Z390 chipset) |
| GPU | Zotac GTX 1080Ti 11GB — installed but unused (no passthrough configured) |
| NVMe | 2TB Samsung PCIe — Proxmox OS + VM disks (LVM-thin) |
| HDD 1 | 2TB — hdd-storage pool, VM image files |
| HDD 2 | 8TB — media8tb storage pool, media server virtual disk |
| NIC | Intel I219V onboard 1GbE |

---

## Proxmox

- **Version:** Proxmox VE 9.1
- **Storage pools:**
  - `local-lvm` — NVMe, LVM-thin, supports linked clones and snapshots
  - `hdd-storage` — 2TB HDD, qcow2 image files
  - `media8tb` — 8TB HDD, media server virtual disk
- Web UI on port 8006

---

## VM / LXC Summary

| ID | Name | Type | Purpose |
|----|------|------|---------|
| 100 | WinVMGeneral | VM | Windows 11 general-purpose |
| 101 | minecraft | LXC | Minecraft server |
| 102 | factorio | LXC | Factorio server (OFSM) |
| 103 | satisfactory | LXC | Satisfactory dedicated server |
| 104 | win11template | VM | Win11 linked clone template |
| 105 | mediaserver | VM | Jellyfin media stack |

All containers and VMs are set to auto-start on Proxmox boot.

---

## LED Control

Motherboard has MSI Mystic Light RGB. Since the host is headless, OpenRGB 1.0rc2 runs at boot via systemd to kill all LEDs.

```bash
# Manual override (SSH into host)
QT_QPA_PLATFORM=offscreen openrgb --noautoconnect -d 0 -m Direct -c 000000

# Service status
systemctl status rgb-off.service
```
