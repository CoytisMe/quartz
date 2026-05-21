---
category: work-note
tags:
  - powershell
  - local-accounts
  - security
  - scheduled-tasks
date: 2026-05-21
publish: true
---
# Restrict Local Account Logon Hours

Restrict non-domain accounts from logging in outside business hours using a scheduled task.

## Script

```powershell
$hour = (Get-Date).Hour
$day  = (Get-Date).DayOfWeek

$adminSids = @("S-1-5-32-544") # Local Administrators group SID

$nonAdminUsers = Get-LocalUser | Where-Object {
    $_.Enabled -ne $null -and
    (Get-LocalGroupMember -SID $adminSids[0] -ErrorAction SilentlyContinue).Name -notcontains "$env:COMPUTERNAME\$($_.Name)"
}

if ($day -ne 'Saturday' -and $day -ne 'Sunday' -and ($hour -ge 6 -and $hour -lt 21)) {
    $nonAdminUsers | Enable-LocalUser
} else {
    $nonAdminUsers | Disable-LocalUser
}
```
Checks that the time is the within the specified hours and the day isn't a weekend. If it's not it runs a loop to target and disable all non admin accounts.

The script to create this as a .ps1 in Program Data remotely can be found below.

## Scheduled Task Setup

Create a single task with two triggers:
- **On schedule** — Runs hourly to make sure it doesn't get caught out
- **At startup** — handles cases where the machine was off when a trigger fired


The startup trigger ensures the account state is correct regardless of when the machine was last on — the script checks the current time and enables or disables accordingly.

### Script for that
```powershell
$scriptPath = "C:\ProgramData\Scripts\Set-LocalAccountHours.ps1"

$action = New-ScheduledTaskAction -Execute "powershell.exe" `
    -Argument "-ExecutionPolicy Bypass -NonInteractive -File `"$scriptPath`""

$triggerStartup = New-ScheduledTaskTrigger -AtStartup
$triggerHourly  = New-ScheduledTaskTrigger -Once -At (Get-Date) `
    -RepetitionInterval (New-TimeSpan -Hours 1)

$settings   = New-ScheduledTaskSettingsSet -StartWhenAvailable -ExecutionTimeLimit (New-TimeSpan -Minutes 5)
$principal  = New-ScheduledTaskPrincipal -UserId "SYSTEM" -RunLevel Highest

Register-ScheduledTask -TaskName "Set-LocalAccountHours" `
    -Action $action `
    -Trigger $triggerStartup, $triggerHourly `
    -Settings $settings `
    -Principal $principal `
    -Force
```

## Create the script via RMM etc
```powershell
$dir = "C:\ProgramData\Scripts"
if (-not (Test-Path $dir)) { New-Item -ItemType Directory -Path $dir | Out-Null }

$content = @'
$hour = (Get-Date).Hour
$day  = (Get-Date).DayOfWeek

$adminSids = @("S-1-5-32-544")

$nonAdminUsers = Get-LocalUser | Where-Object {
    $_.Enabled -ne $null -and
    (Get-LocalGroupMember -SID $adminSids[0] -ErrorAction SilentlyContinue).Name -notcontains "$env:COMPUTERNAME\$($_.Name)"
}

if ($day -ne 'Saturday' -and $day -ne 'Sunday' -and ($hour -ge 6 -and $hour -lt 21)) {
    $nonAdminUsers | Enable-LocalUser
} else {
    $nonAdminUsers | Disable-LocalUser
}
'@

Set-Content -Path "$dir\Set-LocalAccountHours.ps1" -Value $content -Encoding UTF8
```