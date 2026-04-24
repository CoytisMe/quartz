---
category: script
tags:
  - powershell
  - exchange
  - focused-inbox
publish: true
---

## One Guy
```powershell
set-focusedinbox -identity "user@company.com.au" -focusedinboxon $true
```

## Whole Place
```powershell
Set-OrganizationConfig -FocusedInboxOn $false
```

