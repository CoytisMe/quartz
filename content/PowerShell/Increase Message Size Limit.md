---
category: script
tags:
  - powershell
  - exchange
  - attachments
  - message-size
publish: true
---

Need bigger attachments?

Can be done individually in the EAC or or just the below command to do the whole org.

~~~powershell
Get-Mailbox -Resultsize Unlimited | Set-Mailbox -MaxReceiveSize 50MB -MaxSendSize 50MB
~~~