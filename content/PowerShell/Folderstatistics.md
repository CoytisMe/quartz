---
category: script
tags:
  - powershell
  - exchange
  - mailbox
  - folder-statistics
publish: true
---
## Find Conflicts in FolderStatistics

```powershell
Get-MailboxFolderStatistics -Identity  | Where {$_.FolderType -eq "Conflicts"} | Select-Object FolderSize
```