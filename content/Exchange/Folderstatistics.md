---
category: tech
date: 2026-06-29
publish: true
tags:
  - powershell
  - exchange
  - mailbox
  - folder-statistics
---
## Find Conflicts in FolderStatistics

```powershell
Get-MailboxFolderStatistics -Identity  | Where {$_.FolderType -eq "Conflicts"} | Select-Object FolderSize
```

also... [[Script Commands]]
