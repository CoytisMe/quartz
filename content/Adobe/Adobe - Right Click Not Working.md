---
category: troubleshooting
tags:
  - adobe
  - acrobat
  - regedit
  - registry
publish: true
---
https://community.adobe.com/t5/acrobat-reader-discussions/no-longer-able-to-change-scale-ratio-on-acrobat-reader/td-p/14608464/page/2

### Regedit
**32 bit:** 

- Computer\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Policies\Adobe\Adobe Acrobat\DC\FeatureLockDown\

**64 bit:**

- Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Adobe\Adobe Acrobat\DC\FeatureLockDown\

Registry : bNewMegaverbRCMExp(REG_DWORD) = 1 (to disable)


![[Pasted image 20250114092439.png]]