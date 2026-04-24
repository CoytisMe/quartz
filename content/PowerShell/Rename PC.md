---
category: script
tags:
  - powershell
  - windows
  - rename
publish: true
---
```powershell
Rename-Computer -NewName (Read-Host "Enter Name")
```

	-restart 
at the end if you want it to restart