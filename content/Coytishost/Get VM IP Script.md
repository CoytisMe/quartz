---
category: tech
date: 2026-10-12
publish: true
tags:
  - coytishost
  - proxmox
  - bash
  - homelab
source:
---
## I have a script to make temp VMs
I don't know if it's covered in here.
But with the 2 VLANS I can't remote into them by name, I put Temp101, Temp102 etc in my hosts file, but I need to know what their IP is.


## Script to get the IP of a VM

```bash
#!/bin/bash
read -p "Which VM? " VM
IP=$(qm guest exec "$VM" -- ipconfig | grep -oP 'IPv4 Address[. ]*: \K[0-9.]+' | grep -v '^169\.254\.' | head -n1)

if [ -n "$IP" ]; then
    echo "VM $VM is on $IP"
else
    echo "Couldn't get an IP for VM $VM (is it running, with the guest agent installed?)"
fi
```

Related: [[Proxmox Terminal Commands]] (the other `qm` one-liners), [[Tailscale auto-enroll for temp VMs]] (same `qm guest exec` + qemu-guest-agent pattern against the Windows template), [[my disposable windows]]

## Followup
- Windows guests only: it runs `ipconfig` inside the guest. A Linux VM would need `ip -4 addr` instead, or skip the exec entirely with `qm guest cmd <vmid> network-get-interfaces`, which works for any guest with the agent running.
- `qm guest exec` returns JSON with the output in `out-data` (escaped `\r\n`), so the grep is matching inside that JSON blob. Works, just worth knowing if it ever stops matching.
- Could be a one-liner section in [[Proxmox Terminal Commands]] rather than its own page. Left separate with `publish: false` so nothing new goes live on howto.coytis.me without you looking.
