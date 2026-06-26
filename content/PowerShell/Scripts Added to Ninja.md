---
category: how-to
date:
publish: false
tags:
  - work
  - powershell
  - scripts
---
### All passwords never expire

```powershell
Rename-Computer -NewName $env:newname
```
### Rename Computer 
w/ variable for name
```powershell
Get-LocalUser | Where-Object {$_.Enabled -eq $true} | Set-LocalUser -PasswordNeverExpires $true
```