---
category: work-note
tags:
  - powershell
  - spellbook
date: 2026-05-16
source: "[[16-05-26]]"
publish: true
---
# Powershell Spellbook Setup

Running Powershell from the git clone in C: instead of PSGallery

Means the modules runs in new tabs but does load the exchange and graph modules until they're needed.

```ps
cd spellbook
git pull
```

Link to the script in the PS profile
```
Import-Module C:\Users\Rick\Spellbook\Spellbook.psm1
```
