---
category: how-to
publish: true
tags:
  - outlook
  - microsoft
  - exchange
  - calendar
date: 2026-04-16
---
# Automatic Calendar Events from Outlook
## Turning off "Events from Email"
Tell Microsoft they're not helping
### Single Person
#### By Hand

[outlook.office.com](https://outlook.office.com/)

Settings (The Cog) > Calendar > Events from email > Turn all off

![[Automatic Calendar Events from Outlook.png]]

#### The powershell way
```powershell
Set-MailboxCalendarConfiguration -Identity "user@domain.com" -EventsFromEmailEnabled $false
```

### The Whole Org
#### There is only one way
```powershell
Get-Mailbox -ResultSize Unlimited | Set-MailboxCalendarConfiguration -EventsFromEmailEnabled $false
```