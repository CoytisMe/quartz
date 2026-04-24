---
category: script
tags:
  - powershell
  - exchange
  - archive
  - mailbox
publish: true
---
Archive will automatically expand by 10gb when it reaches capacity

## Required Licences
- Exchange Online (Plan 2) - $13
	- The setting will activate with an EOP1 but it won't expand past 110gb
- Exchange Online Archiving ($5)

**Clients will still receive notifications that they're archive are about to fill up.**

## See [[Check Archive Mailbox Size]] 

## How to check

## One Person
```powershell
Enable-Mailbox <user mailbox> -AutoExpandingArchive
```

```powershell
Get-Mailbox <user mailbox> | FL AutoExpandingArchiveEnabled
```

## Whole place

```powershell
Set-OrganizationConfig -AutoExpandingArchive
```
