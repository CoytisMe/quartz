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

## Notes

- SSH config is per-machine — each PC can point the same alias (`VMhost`) at a different IP if needed, while the `.cmd` scripts in OneDrive stay identical across both PCs.
- No need for local admin when SSHing — permissions are determined by the remote user account.
