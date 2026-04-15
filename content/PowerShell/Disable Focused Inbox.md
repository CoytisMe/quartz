---
category: script
tags:
  - powershell
  - exchange
  - focused-inbox
publish: true
---
## Whole Place
```powershell
Set-OrganizationConfig -FocusedInboxOn $false
```

## One Guy
```powershell
set-focusedinbox -identity "user@company.com.au" -focusedinboxon $true
```