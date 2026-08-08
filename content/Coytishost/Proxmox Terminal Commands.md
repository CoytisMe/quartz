---
category: reference
date: 2026-07-01
publish: true
tags:
---
## Virtual Machines
List VMs
```bash
qm list
```

Start a VM
```bash
qm start <vmid>
```

Shutdown a VM (nicely)
```bash
qm shutdown <vmid>
```

Shutdown a VM (with both hands)
```bash
qm stop <vmid>
```

Restart a VM (nicely)
```bash
qm reboot <vmid>
```

Restart a VM (Both Hands)
```bash
qm reset
```

Delete a VM
```bash
qm destroy <vmid>
```

Show a VMs config file
```bash
qm config <vmid>
```

Open a direct terminal
```bash
qm terminal <vmid>
```

## LXC Containters
The following are the same as above just swap 'qm' for 'pct'
- List
- Start
- Stop
- Shutdown
- Config

Open a root shell
```bash
pct enter <ctid>
```

Run a command on the container (Don't ask me what they are)
```bash
pct exec <ctid> -- command
```

## Other
Show status and usage of all disks
```bash
pvesm status
```

Show versions of all installed packages
```bash
pveverion -v
```

Restart web management service
```bash
systemctl restart pvedaemon
```

Show real time system logs
```bash
journalctl -f
```
