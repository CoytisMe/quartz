---
category: personal
tags:
  - archery
  - ndi
  - raspberry-pi
  - target-cam
date: 2026-05-26
publish: true
---
I'd publish the build log but it's got sooooo much personal info it in so fuck off

Goal: USB-powered, WiFi NDI broadcaster at 1280x720 @ 15–30fps. Physically tiny, single USB charger into the wall.

Existing NUC setup documented at [[log - target cam nuc]] — the software stack is identical, just compiled for ARM instead of x86_64.

---

## Pi Options

### Pi Zero 2W — Smallest possible
- Quad-core Cortex-A53 @ 1GHz, 512MB RAM
- **Power:** micro USB (PWR IN port) — any USB charger works, draws ~1.5–2W under load
- **Webcam:** USB OTG adapter into the second micro USB port
- **WiFi:** built-in
- **Size:** credit card, genuinely tiny
- **Concern:** 1GHz is tight for 720p @ 30fps NDI. Likely fine at **15fps**, worth testing at 30fps — the YUYV→UYVY byte swap is cheap but NDI has overhead
- **Cost:** ~$20 AUD

### Pi 3B+ — Middle ground
- Quad-core Cortex-A53 @ 1.4GHz, 1GB RAM
- **Power:** micro USB, draws ~2–3W — needs a decent charger (2.5A+)
- **Webcam:** full-size USB-A port, no adapter needed
- **WiFi:** built-in
- **720p @ 30fps:** should handle it comfortably
- **Cost:** ~$50–60 AUD (harder to find new)

### Pi 4 (2GB) — Safe bet
- Quad-core Cortex-A72 @ 1.8GHz, 2GB RAM
- **Power:** USB-C, draws ~3–4W — needs a 5V/3A USB-C charger
- **Webcam:** full-size USB-A port
- **WiFi:** built-in
- **720p @ 30fps:** no issues, headroom to spare
- **Cost:** ~$60–80 AUD

---

## Recommendation

**Start with Pi Zero 2W** if you want the smallest possible footprint — test at 15fps first, try 30fps and see. If it struggles, Pi 4 is the reliable fallback.

For the power setup on Zero 2W:
- USB charger → PWR IN (left micro USB port)
- Webcam → micro USB OTG adapter → USB port (right micro USB port)
- One cable in the wall, one cable to the webcam, done

---

## Setup — Same as NUC, ARM Version

OS: **Raspberry Pi OS Lite (64-bit)** — headless, no desktop needed. Configure WiFi during flash via Raspberry Pi Imager (Advanced Options).

### 1. Install build tools
```bash
sudo apt update && sudo apt install -y curl gcc make v4l-utils
```

### 2. Install NDI SDK (ARM64)
```bash
wget https://downloads.ndi.tv/SDK/NDI_SDK_Linux/Install_NDI_SDK_v6_Linux.tar.gz -O /tmp/ndi-sdk.tar.gz
cd /tmp && tar -xzf ndi-sdk.tar.gz
yes | PAGER=cat sh /tmp/Install_NDI_SDK_v6_Linux.sh

# ARM64 — note aarch64 instead of x86_64
sudo cp -P ~/NDI\ SDK\ for\ Linux/lib/aarch64-linux-gnu/* /usr/local/lib/
sudo cp ~/NDI\ SDK\ for\ Linux/include/* /usr/local/include/
sudo ldconfig
sudo ln -sf /usr/local/lib/libndi.so.6 /usr/local/lib/libndi.so.5
```

### 3. Compile ndi-webcam
Same C source as the NUC (see [[log - target cam nuc]]). Change `NDI_NAME` at the top to something like `"Pi Target Cam"`.

```bash
gcc -O2 -Wall -I/usr/local/include -o /tmp/ndi-webcam /tmp/ndi-webcam.c -L/usr/local/lib -lndi -Wl,-rpath,/usr/local/lib
sudo install -m 755 /tmp/ndi-webcam /usr/local/bin/ndi-webcam
```

### 4. Systemd service
Same service file as the NUC — copy it over, then:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now ndi-webcam
```

### 5. Set a static IP
Either via your router's DHCP reservation (easiest) or via `/etc/dhcpcd.conf`:
```
interface wlan0
static ip_address=192.168.1.XX/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

---

## FPS Tweak

To drop to 15fps, edit the `#define FPS 30` line in the C source to `15`, recompile, reinstall, restart service. Halves the NDI bandwidth and CPU load.

---

## Notes
- If the webcam doesn't appear at `/dev/video0` run `v4l2-ctl --list-devices` to find the right path
- NDI discovery is multicast — Pi and receiving machine need to be on the same subnet/WiFi network
- If multicast is blocked, NDI Tools on the receiving end has a manual source entry option
