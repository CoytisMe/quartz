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
set-focusedinbox -identity (Read-Host "Enter mailbox") -focusedinboxon $false
```

## Whole Place
```powershell
Set-OrganizationConfig -FocusedInboxOn $false
```

