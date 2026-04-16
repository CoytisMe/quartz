---
category: reference
tags:
  - powershell
  - exchange
  - work
date: 2026-03-26
publish: true
---
# Check In-Place Archive Mailbox Size

## Single Mailbox
**Archive Used**
```powershell
Get-MailboxStatistics user@domain.com -Archive | Select DisplayName, TotalItemSize, ItemCount
```

**Archive Max**
```powershell
Get-Mailbox user@domain | Select ArchiveQuota, ArchiveWarningQuota
```

## All Archive Mailboxes

```powershell
Get-Mailbox -ResultSize Unlimited | Get-MailboxStatistics -Archive | Select DisplayName, TotalItemSize | Sort TotalItemSize -Descending
```

## Notes
- The `-Archive` switch is required — without it, the command returns primary mailbox stats instead of the archive.
- Must be connected to Exchange Online PowerShell first: `Connect-ExchangeOnline`


Get-MailboxStatistics user@domain -Archive | Select DisplayName, TotalItemSize, ItemCount