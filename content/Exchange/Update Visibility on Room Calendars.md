---
category: how-to
date: 2026-08-03
publish: true
tags:
  - exchange
  - microsoft
  - powershell
  - calendar
---
### Connect-ExchangeOnline
```powershell
Set-MailboxFolderPermission -Identity "Boardroom@yourdomain.com:\Calendar" -User Default -AccessRights Editor
```