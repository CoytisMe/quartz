---
category: work-note
tags:
  - powershell
  - exchange
  - graph
  - tips
  - scripts
date: 2026-04-23
publish: true
---
# PowerShell Script Tips & Tricks
#### I think most of this is doubled up in other notes but fuck it we publish.
Patterns and techniques for making scripts more reusable and user-friendly.

---

## Ask for input instead of hardcoding

Use `Read-Host` to prompt for a value at runtime — no more swapping emails in Notepad.

```powershell
$mailbox = Read-Host "Enter mailbox"
```

Or inline:

```powershell
Get-Mailbox -Identity (Read-Host "Enter mailbox") | Select ArchiveQuota
```

---

## Auto-connect to Exchange Online

Add this to the top of any EXO script. Connects only if not already connected.

```powershell
if (-not (Get-ConnectionInformation)) {
    Connect-ExchangeOnline
}
```

Requires: `ExchangeOnlineManagement` module

---

## Auto-connect to Microsoft Graph

Add this to the top of any Graph script. Connects only if not already connected.

```powershell
if (-not (Get-MgContext)) {
    Connect-MgGraph -Scopes "UserAuthenticationMethod.ReadWrite.All"
}
```

Requires: `Microsoft.Graph.Identity.SignIns` module

Check if installed:
```powershell
Get-Module Microsoft.Graph.Identity.SignIns -ListAvailable
```

Install if missing:
```powershell
Install-Module Microsoft.Graph.Identity.SignIns
```

Check if connected to MgGraph
```powershell
Get-MgContext
```

---

## Environment Variables in Paths

PowerShell does **not** expand CMD-style environment variables like `%USERPROFILE%` inside strings. That syntax only works in Command Prompt and batch files.

In PowerShell, environment variables are accessed via `$env:VARIABLENAME`:

```powershell
# Wrong — PowerShell treats this as a literal string
"$%USERPROFILE%\Desktop\file.csv"

# Right
"$env:USERPROFILE\Desktop\file.csv"
```

**Common ones:**

| CMD | PowerShell |
|-----|------------|
| `%USERPROFILE%` | `$env:USERPROFILE` |
| `%APPDATA%` | `$env:APPDATA` |
| `%TEMP%` | `$env:TEMP` |
| `%COMPUTERNAME%` | `$env:COMPUTERNAME` |
| `%USERNAME%` | `$env:USERNAME` |

Example — export CSV to desktop regardless of who's logged in:

```powershell
$results | Export-Csv -Path "$env:USERPROFILE\Desktop\LicensedUsers.csv" -NoTypeInformation
```

This works on any machine without hardcoding `C:\Users\Rick\...`

---

## Write-Host — printing messages to the terminal

Use `Write-Host` to print status messages, section headers, or confirmations. It outputs directly to the terminal and doesn't affect the pipeline.

```powershell
# Basic message
Write-Host "Done."

# With a leading blank line for readability
Write-Host "`nDone."

# With a variable
Write-Host "`nExported to $env:USERPROFILE\Desktop\LicensedUsers.csv"
```

The backtick `` ` `` is PowerShell's escape character. `` `n `` is a newline — handy for spacing output between sections.

---

## Force immediate output formatting

PowerShell sometimes buffers pipeline output and prints it out of order. Add `| Format-Table` to force it to print immediately.

```powershell
Get-MailboxStatistics $mailbox -Archive | Select DisplayName, TotalItemSize, ItemCount | Format-Table
```

---

## Comparison Operators

PowerShell can't use `>`, `<`, `>=` etc. directly in conditionals — it uses text-based operators instead:

| Operator | Meaning |
|----------|---------|
| `-eq` | Equal to |
| `-ne` | Not equal to |
| `-gt` | Greater than |
| `-ge` | Greater than or equal to |
| `-lt` | Less than |
| `-le` | Less than or equal to |

```powershell
if ($hour -ge 8 -and $hour -lt 18) { ... }
```

Also useful for day-of-week checks — `DayOfWeek` returns the full day name as a string:

```powershell
$day = (Get-Date).DayOfWeek

if ($day -ne 'Saturday' -and $day -ne 'Sunday') { ... }
```

---

## Read-Host and quoted paths (from Copy Path)

Windows Explorer's right-click → **Copy Path** wraps the path in literal double quotes (e.g. `"C:\Users\Rick\Downloads\file.exe"`). `Read-Host` doesn't strip them — they get stored as part of the string, quote characters included. Passing that straight into a native command (`scp`, `robocopy`, etc.) fails, because it's now trying to find a file whose name literally includes `"` characters.

Fix — trim the quotes, then re-quote when calling the command (so paths with spaces still work):

```powershell
$file = Read-Host "Enter file path"
$file = $file.Trim('"')
scp "$file" user@host:/remote/path/
```

`.Trim('"')` strips a leading/trailing `"` if present and does nothing if you paste a path without quotes — safe either way.

---

## Check when a transport rule was last modified

`Get-TransportRule` doesn't show change history by default, but `WhenChanged` on the rule object tells you the last modification time.

```powershell
Get-TransportRule "Insert Rule Name" | FL Name, WhenChanged
```

Useful for confirming whether a rule was actually touched recently, e.g. when troubleshooting mail flow changes someone swears they didn't make.

---

## Scripts

### checkarchive.ps1
Check archive size and quota for a mailbox.

```powershell
if (-not (Get-ConnectionInformation)) {
    Connect-ExchangeOnline
}

$mailbox = Read-Host "Enter mailbox"

Write-Host "`n--- Archive Stats ---"
Get-MailboxStatistics $mailbox -Archive | Select DisplayName, TotalItemSize, ItemCount | Format-Table

Write-Host "`n--- Archive Quota ---"
Get-Mailbox -Identity $mailbox | Select ArchiveQuota, ArchiveWarningQuota, AutoExpandingArchiveEnabled | Format-Table
```

---

### newtap.ps1
Generate a Temporary Access Pass for a user.

```powershell
if (-not (Get-MgContext)) {
    Connect-MgGraph -Scopes "UserAuthenticationMethod.ReadWrite.All"
}

New-MgUserAuthenticationTemporaryAccessPassMethod -UserId (Read-Host "Enter UPN") -LifetimeInMinutes 60 | Select TemporaryAccessPass
```

---

### removetaps.ps1
Remove all Temporary Access Passes for a user.

```powershell
if (-not (Get-MgContext)) {
    Connect-MgGraph -Scopes "UserAuthenticationMethod.ReadWrite.All"
}

$upn = Read-Host "Enter UPN"
Get-MgUserAuthenticationTemporaryAccessPassMethod -UserId $upn | ForEach-Object {
    Remove-MgUserAuthenticationTemporaryAccessPassMethod -UserId $upn -TemporaryAccessPassAuthenticationMethodId $_.Id
}
```

---

### setpasswordneverexpires.ps1
Set password never expires on all enabled local accounts. Safe to run as system — only affects local accounts, not Entra/domain accounts.

```powershell
Get-LocalUser | Where-Object {$_.Enabled -eq $true} | Set-LocalUser -PasswordNeverExpires $true
```

---

### uploadstuff.ps1
Upload a file to stuff.coytis.me's downloads folder. Paste a Copy Path result straight in — quotes are handled automatically.

```powershell
$file = Read-Host "Enter file path"
$file = $file.Trim('"')
scp "$file" rick@192.168.1.96:/opt/media/downloads/
```

---

### deletestuff.ps1
Remove a file from stuff.coytis.me's downloads folder.

```powershell
$name = Read-Host "Enter filename to delete"
$name = $name.Trim('"')
ssh rick@192.168.1.96 "rm /opt/media/downloads/$name"
```

---

### deletedoublejsons.ps1
Removes duplicate `workspace*.json` files from the vault's `.obsidian` folder created by OneDrive sync conflicts. Located in Scripts / Other folder.

```powershell
$files = Get-ChildItem -Path "C:\Users\Rick\OneDrive - Coytis\Obsidian\Rick\.obsidian" -Filter workspace*.json | Where-Object Name -ne "workspace.json"
$files | Remove-item
```

*Followup: treats the symptom, not the cause — if these conflicts keep recurring, worth checking whether Obsidian sync/OneDrive settings can reduce them, or scheduling this to run automatically on login.*

---

## Notes

- Scripts live at `C:\Users\Rick\Scripts` — symlinked to `OneDrive - Coytis\Scripts` so they sync across both PCs automatically
- Always use `Read-Host` instead of hardcoded emails/UPNs in scripts that will be reused or published in KBs
- Connection checks mean scripts are safe to run standalone or after an existing session — no double auth prompts
