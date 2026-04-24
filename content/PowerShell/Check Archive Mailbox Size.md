---
category: reference
tags:
  - powershell
  - exchange
  - work
date: 2026-03-26
publish: true
---
# Check Archive Mailbox Size

## Single Mailbox
**Archive Used**
```powershell
Get-MailboxStatistics user@domain.com -Archive | Select DisplayName, TotalItemSize, ItemCount
```

**Archive Max**
```powershell
Get-Mailbox -Identity "user@domain.com" | Select ArchiveQuota, ArchiveWarningQuota, AutoExpandingArchiveEnabled
```

## All Archive Mailboxes

```powershell
Get-Mailbox -ResultSize Unlimited | Get-MailboxStatistics -Archive | Select DisplayName, TotalItemSize | Sort TotalItemSize -Descending
```

## Very Related to [[AutoExpandingArchive]]
## Notes
- The `-Archive` switch is required — without it, the command returns primary mailbox stats instead of the archive.
- Must be connected to Exchange Online PowerShell first: `Connect-ExchangeOnline`


