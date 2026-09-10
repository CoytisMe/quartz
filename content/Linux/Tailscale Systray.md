---
category: tech
date: 2026-09-06
publish: true
tags:
  - linux
  - tailscale
source: "[[05-09-26]]"
---
## Tailscale Systray on Linux
Official one.

Systray is the thing near the clock so you can actually interact with Tailscale

To start:
```bash
tailscale --systray
```

To set autostart:
```bash
tailscale configure systray --enable-startup=systemd
```
