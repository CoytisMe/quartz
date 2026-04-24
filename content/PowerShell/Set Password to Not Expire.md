---
category: script
tags:
  - powershell
  - windows
  - users
  - password
publish: true
---
### One User
```powershell
Set-LocalUser -Name (Read-Host "Enter User") -PasswordNeverExpires $true
```
### All Local Users
```powershell
Get-LocalUser | Set-LocalUser -PasswordNeverExpires $true
```
### Currently Logged In User
```powershell
Set-LocalUser -Name $env:USERNAME -PasswordNeverExpires $true
```