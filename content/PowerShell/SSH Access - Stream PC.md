---
category: tech
date: 2026-08-08
publish: false
tags:
  - ssh
  - stream-pc
  - claude-code
---

# SSH Access — Stream PC

Built 2026-08-08. Lets the Main PC hold a Claude Code session on the Stream PC from across the desk, without switching inputs.

## Connect

```bash
ssh Rick@192.168.2.99
```

Then `claude` once you're in.

| | |
|---|---|
| Host | `rickstreampc` |
| IP | `192.168.2.99` (Ethernet 2, /24) |
| Tailscale IP | `100.67.238.104` |
| User | `Rick` — **local** account, so no `MicrosoftAccount\` prefix |
| Password | Normal Windows password |
| Landing shell | Windows PowerShell 5.1 |

`ssh Rick@rickstreampc` works too if NetBIOS resolves. Fall back to the IP if not.

## How it's configured

- **Capability:** `OpenSSH.Server~~~~0.0.1.0`, installed via `dism.exe` (see *Gotchas* — the PowerShell route was broken)
- **Service:** `sshd`, startup Automatic
- **Firewall rule:** `OpenSSH-Server-In-TCP` — TCP/22, **Private profile only**, `RemoteAddress` restricted to `192.168.2.0/24` and `100.64.0.0/10`
- **Host keys:** ed25519 / ecdsa / rsa, generated on first service start
- **Auth:** password *and* pubkey both enabled. Password auth is deliberate — it's LAN-scoped and this is a convenience box, not an exposed one.
- **DefaultShell:** `HKLM:\SOFTWARE\OpenSSH\DefaultShell` → `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

The daemon binds `0.0.0.0:22` and `[::]:22`. **The firewall is what limits exposure, not the bind address.** If that rule gets reset or widened, the box is listening on every interface.

## Watch out for

**Network profile flipping to Public.** The rule is Private-only. If Windows re-detects the network (new router, adapter swap) and marks it Public, SSH silently stops accepting connections. Check with:

```powershell
Get-NetConnectionProfile | Select-Object InterfaceAlias, NetworkCategory
```

Fix: `Set-NetConnectionProfile -InterfaceAlias 'Ethernet 2' -NetworkCategory Private`

**Subnet change.** If the LAN ever moves off `192.168.2.0/24` the rule needs updating:

```powershell
Set-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' `
  -RemoteAddress @('192.168.X.0/24','100.64.0.0/10')
```

**Dropped connections lose the TUI.** No tmux on native Windows. Reconnect and run `claude --continue` (or `--resume` to pick a session) — the conversation survives, the terminal doesn't.

**Browser tools won't work over SSH.** The Claude in Chrome MCP tools need the interactive desktop session. Over SSH they'll either fail or drive the Chrome on the console session. Do browser work locally.

## If you want keys instead of a password

Rick is an admin, so the key does **not** go in `~/.ssh/authorized_keys` — Windows OpenSSH ignores that for admin accounts. It goes in:

```
C:\ProgramData\ssh\administrators_authorized_keys
```

ACLs must be **Administrators and SYSTEM only** or sshd silently refuses it:

```powershell
icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r
icacls C:\ProgramData\ssh\administrators_authorized_keys /grant "Administrators:F" /grant "SYSTEM:F"
```

Then set `PasswordAuthentication no` in `C:\ProgramData\ssh\sshd_config` and `Restart-Service sshd`.

## Gotchas hit during setup

**`Get-WindowsCapability` throws "Class not registered" — but only in PowerShell 7.** This is *not* machine corruption. The Dism module is a Windows PowerShell module; PS7 loads it through a compatibility shim and the COM activation fails there. It works fine in `powershell.exe` (5.1).

Three ways round it:

```powershell
# 1. use dism.exe
dism /online /get-capabilities /format:table

# 2. shell out to 5.1
powershell.exe -NoProfile -Command "Get-WindowsCapability -Online -Name OpenSSH*"

# 3. load the module through the compat shim inside pwsh 7
Import-Module Dism -UseWindowsPowerShell
```

Applies to the whole Dism module, so expect it from `Get-WindowsOptionalFeature`, `Add-WindowsCapability` etc. too.

**pwsh 7 is the Store/MSIX build** with a version-stamped path, which is why `DefaultShell` points at 5.1 rather than 7. See [[PowerShell 7 MSIX to MSI Swap]] — after that swap, repoint `DefaultShell` to `C:\Program Files\PowerShell\7\pwsh.exe`.

## Verify it's healthy

```powershell
Get-Service sshd
Get-NetTCPConnection -LocalPort 22 -State Listen
Get-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' | Get-NetFirewallAddressFilter
& 'C:\Windows\System32\OpenSSH\sshd.exe' -T | Select-String 'passwordauthentication|pubkeyauthentication'
```

Auth failures log to Event Viewer under `Applications and Services Logs → OpenSSH → Operational`.
