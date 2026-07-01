---
category: tech
tags:
  - adobe
  - acrobat
  - security
  - fix
date: 2026-06-30
source: "[[Dailys/2026/06/30-06-26]]"
publish: true
---
# Adobe Save As Issue — Protected Mode Fix

Symptom: Adobe always prompting "Save As" (wouldn't save in place).

## Fix

Go to **Preferences > Security (Enhanced)** and:

- Uncheck **Enable protected mode at startup**
- Uncheck **Enable Enhanced Security**
- Add `C:\` as a trusted folder path

May not have needed the last two steps but did them anyway.

## See Also

- [[Adobe Reader Mode Fix]]

## Followup

Protected Mode restricts write access as a sandbox measure — which is why save-in-place breaks. Worth noting: in a domain environment, these settings can be re-enforced by GPO (Adobe ADMX templates), so if this fix reverts, check GPO before doing it manually again. The trusted path approach is the cleaner long-term fix if GPO isn't involved.
