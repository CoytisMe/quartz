---
category: tech
date: 2026-06-29
publish: true
tags:
  - powershell
  - exchange
  - attachments
  - message-size
---

Need bigger attachments?

Can be done individually in the EAC or just the below command to do the whole org.

~~~powershell
Get-Mailbox -Resultsize Unlimited | Set-Mailbox -MaxReceiveSize 50MB -MaxSendSize 50MB
~~~

also... [[Script Commands]]
