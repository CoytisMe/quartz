---
category: how-to
publish: true
tags:
  - leap
  - cmd
  - adobe
  - acrobat
  - troubleshooting
date: 2026-04-16
---
### Update / Reinstall Adobe Add-in
Useful script for LEAP's bi-monthly Adobe fuckup.

Force closes LEAP and Adobe and runs LEAPs own add-in installer from ProgramData.

You can also just go to
`C:\ProgramData\LEAP Office\Cloud\Extras\Acrobat Extras
and double click InstallLauncher.exe

```cmd
@echo off
setlocal

echo Closing Adobe and LEAP applications...

taskkill /F /IM AcroRd32.exe 2>NUL
taskkill /F /IM Acrobat.exe 2>NUL
taskkill /F /IM "LEAP Desktop.exe" 2>NUL
taskkill /F /IM "LEAP Accounting.exe" 2>NUL
taskkill /F /IM leapsystray.exe 2>NUL

echo Waiting for processes to close...
timeout /t 3 /nobreak >NUL

echo Running LEAP Acrobat installer...
start "" "C:\ProgramData\LEAP Office\Cloud\Extras\Acrobat Extras\InstallLauncher.exe"

endlocal
```