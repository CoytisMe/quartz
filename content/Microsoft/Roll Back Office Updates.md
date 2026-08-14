---
category: tech
date: 2026-08-12
publish: true
tags:
  - office
  - updates
  - rollback
  - clicktorun
---
## Office Updated?
Caused Shit?

1. Close it all down.
2. Go **[here](https://learn.microsoft.com/en-us/officeupdates/update-history-microsoft365-apps-by-date)** and pick and old update.
3. Write down this number
   ![[Roll Back Office Updates-1.png]]
4. Open CMD as admin and do these commands
```cmd
cd %ProgramFiles%\Common Files\Microsoft Shared\ClickToRun\
```
   
```cmd
OfficeC2RClient.exe /update user updatetoversion=16.0.NUMBERFROMABOVE
```

Office will update.

### Yay

To stop updates landing this often in the first place, see [[Change M365 Update Channel]].

