---
category: tech
date: 2026-08-08
publish: false
tags:
  - powershell
  - ssh
  - stream-pc
---

# PowerShell 7 — Swap Store (MSIX) build for MSI build

**Machine:** Stream PC (192.168.2.99)
**Why:** Current pwsh 7 is the Microsoft Store / MSIX build, installed at
`C:\Program Files\WindowsApps\Microsoft.PowerShell_7.6.4.0_x64__8wekyb3d8bbwe\pwsh.exe`

That path has the **version number baked into it**, so it changes on every PowerShell update. Anything hardcoding it — sshd's `DefaultShell`, scheduled tasks, shortcuts — breaks silently on the next update. The MSI build installs to the stable `C:\Program Files\PowerShell\7\pwsh.exe` and never moves.

Not urgent. SSH works fine without this.

---

## Before you start

**You cannot uninstall an MSIX package while it's running.** Do this from a **Windows PowerShell 5.1** window (`powershell.exe`), not from pwsh, and not from a Claude Code session — Claude Code runs inside pwsh and will kill its own shell.

Close all pwsh 7 windows first.

## The swap

```powershell
winget uninstall --id Microsoft.PowerShell
winget install --id Microsoft.PowerShell --scope machine
```

Verify:

```powershell
(Get-Command pwsh).Source
# expect: C:\Program Files\PowerShell\7\pwsh.exe
```

## What survives — no action needed

Both builds read the same user profile paths, so these are untouched:

- `C:\Users\Rick\Documents\PowerShell\profile.ps1`
- `C:\Users\Rick\Documents\PowerShell\Modules\` — `StevesScriptorium`, `PnP.PowerShell`, `ExchangeOnlineManagement`, `Microsoft.Graph.*`, `Microsoft.Online.SharePoint.PowerShell`, `Posh-SSH`

## What to fix up after

Windows Terminal generates its pwsh entry dynamically per install, so the Store entry disappears and a new one appears with a **different GUID**. Re-select pwsh as default profile and redo any font/colour tweaks on that profile. That's the only cleanup.

---

## ⚠️ Ask Claude for this afterwards

Once the MSI build is in, sshd's `DefaultShell` needs repointing from Windows PowerShell 5.1 to pwsh 7. Ask for:

> "Repoint the sshd DefaultShell to the MSI pwsh path"

Or just run it — needs an **elevated** shell:

```powershell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" `
  -Name DefaultShell `
  -Value "C:\Program Files\PowerShell\7\pwsh.exe" `
  -PropertyType String -Force
```

No service restart needed — sshd reads this per-login. Test by opening a fresh SSH session from the Main PC; `$PSVersionTable.PSVersion` should show 7.x on landing instead of 5.1.

**Current value (set during SSH build-out):**
`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

---

## Context

Part of the SSH-into-Stream-PC setup — LAN-scoped sshd so the Main PC can hold a Claude Code session on the Stream PC from across the desk. See [[SSH Access - Stream PC]].
