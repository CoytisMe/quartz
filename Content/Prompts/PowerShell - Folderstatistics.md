---
category: script
tags:
  - powershell
  - exchange
  - mailbox
  - folder-statistics
---
## Find Conflicts in FolderStatistics

```
Get-MailboxFolderStatistics -Identity  | Where {$_.FolderType -eq "Conflicts"} | Select-Object FolderSize
```