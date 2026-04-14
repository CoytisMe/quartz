---
category: script
tags:
  - powershell
  - exchange
  - archive
  - mailbox
---
Archive will automatically expand by 10gb when it reaches capacity

## Required Licences
- Exchange Online (Plan 2) - $13
	- The setting will activate with an EOP1 but it won't expand past 110gb
- Exchange Online Archiving ($5)

**Clients will still still receive notifications that they're archive are about to fill up.**

```
connect-exchangeonline
```

## One Person
```
Enable-Mailbox <user mailbox> -AutoExpandingArchive
```

```
Get-Mailbox <user mailbox> | FL AutoExpandingArchiveEnabled
```

## Whole place

```
Set-OrganizationConfig -AutoExpandingArchive
```
