---
category: reference
tags:
  - powershell
  - windows
  - task-scheduler
---
# Create Wake Timer Task

Creates a Windows Task Scheduler job that wakes the PC daily at 8:30am. Useful for ensuring the machine is awake before scheduled automation runs. Requires PowerShell to be run as Administrator.

```powershell
$action = New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c exit"
$trigger = New-ScheduledTaskTrigger -Daily -At 8:30am
$settings = New-ScheduledTaskSettingsSet -WakeToRun
Register-ScheduledTask -TaskName "Morning Wood" -Action $action -Trigger $trigger -Settings $settings -RunLevel Highest -Force
```

> Note: "Allow wake timers" must be enabled in Windows power settings for this to work.
> Settings → System → Power & Sleep → Additional power settings → Change plan settings → Change advanced power settings → Sleep → Allow wake timers → Enabled
