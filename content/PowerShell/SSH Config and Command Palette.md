---
category: reference
date: 2026-06-11
publish: true
tags:
  - ssh
  - powershell
  - powertoys
  - windows-terminal
  - tools
---

## If you don't think Command Palette is the shit you can get out of my face.

How to SSH into named hosts without typing IPs, and launch connections from the PowerToys command palette.

---

## SSH Config

The SSH config file lets you define named aliases for hosts so you can type `ssh VMhost` instead of `ssh user@192.168.1.10`.

**File location:** `C:\Users\Rick\.ssh\config` — no extension, just `config`.

Create it from terminal if it doesn't exist:
```powershell
New-Item -Path "$HOME\.ssh\config" -ItemType File
notepad "$HOME\.ssh\config"
```

**Example config:**
```
Host VMhost
    HostName 192.168.1.10
    User rick
    Port 22

Host otherbox
    HostName 10.0.0.5
    User admin
```

Once set up, `ssh VMhost` works anywhere — terminal, PowerToys Run, scripts.

> Note: SSH config doesn't support passwords. To avoid password prompts, set up SSH key auth (`ssh-keygen` + copy public key to remote `authorized_keys`). That's a later problem.

---

## Launching SSH from PowerToys Run

Just type the SSH command directly in the palette using the `>` shell prefix:

```
> ssh VMhost
```

This opens a PowerShell window already connected. No setup needed beyond having the SSH config in place.

The terminal it opens won't be Windows Terminal with your theme — it'll be a plain shell — but that doesn't matter for SSH since everything runs on the remote machine anyway. Local admin/theme is irrelevant once you're in.

---

## PATH — What It Is

PATH is a Windows environment variable containing a list of folders. When you type a command name anywhere (PowerToys Run, terminal, Run dialog), Windows searches every folder in PATH until it finds a matching file.

**Check what's in PATH:**
```powershell
$env:PATH -split ';'
```

**Add a folder to PATH (user-level, persists across sessions):**
```powershell
[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";C:\Users\Rick\Scripts", "User")
```

Restart the terminal (and PowerToys) after changing PATH for it to take effect.

---

## .cmd Scripts for PowerToys Run (Partially Working)

You can create `.cmd` files in your Scripts folder (`C:\Users\Rick\OneDrive - Coytis\Claude\Scripts\`, symlinked to `C:\Users\Rick\Scripts`) and they'll be runnable by name from PowerToys Run once Scripts is on PATH.

**Example — `sshvmhost.cmd`:**
```batch
@echo off
wt ssh VMhost
```

This would open Windows Terminal directly into the SSH session. Tested and the file is found (shows in fallback results in the palette) but wasn't appearing as a top-level command result during initial setup — may just need PowerToys to fully reindex. Worth revisiting. The `> ssh VMhost` approach works fine in the meantime.

---

## SSH Host Keys — What They Are & Dealing With Warnings

Every SSH server has a **host key** — a unique cryptographic identity generated once when the server/OS is set up. The first time you connect to a host, your SSH client saves that key's fingerprint to `known_hosts` (`C:\Users\Rick\.ssh\known_hosts`). Every connection after that, it checks the server presents the *same* key — that's what stops someone from silently swapping in a fake server and intercepting your session (a man-in-the-middle attack).

**What triggers the big red "REMOTE HOST IDENTIFICATION HAS CHANGED!" warning:**
The key the server just presented doesn't match what's saved in `known_hosts` for that hostname/IP. Two very different explanations:
1. **Benign** — the server was rebuilt/reinstalled (fresh OS = fresh host keys), restored from a snapshot, or the IP got reassigned to a different machine. Common on home labs/VMs that get rebuilt occasionally.
2. **Actual concern** — someone is intercepting the connection and presenting a different key (MITM). Much less likely on a private home LAN behind your own router, but not impossible if the network itself is compromised.

**Don't just blindly clear `known_hosts` and reconnect — verify the new key is legitimate first.** Ideally check over a path other than the one being questioned (a different device/session, or pulling the key straight from the host if you have another way in, e.g. a Proxmox console for a VM).

Get the live fingerprint a server is currently presenting:
```powershell
ssh-keyscan -t ed25519 <host>
```

Compare that against the fingerprint shown in the warning. To compute a SHA256 fingerprint from a raw base64 key blob yourself (useful if you want to double-check without connecting):
```powershell
python3 -c "
import base64, hashlib
key_b64 = 'PASTE_BASE64_BLOB_HERE'
raw = base64.b64decode(key_b64)
digest = hashlib.sha256(raw).digest()
print('SHA256:' + base64.b64encode(digest).decode().rstrip('='))
"
```
If the fingerprints match exactly, the new key is genuine — safe to proceed.

**Once confirmed legitimate, clear the stale entry:**
```powershell
ssh-keygen -R <hostname-or-ip>
```
Then reconnect as normal — SSH prompts to trust the new key once, and saves it.

**Real example (2026-07-07):** hit this exact warning connecting to the media server (192.168.1.96) after the Omada VLAN migration. Verified via `ssh-keyscan` from a separate, already-trusted session that the new key's fingerprint matched exactly what the warning reported — confirmed it was just a stale `known_hosts` entry (not a MITM) and cleared it safely.

---

## Notes

- SSH config is per-machine — each PC can point the same alias (`VMhost`) at a different IP if needed, while the `.cmd` scripts in OneDrive stay identical across both PCs.
- No need for local admin when SSHing — permissions are determined by the remote user account.
