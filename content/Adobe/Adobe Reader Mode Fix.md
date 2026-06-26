---
category: work-note
tags:
  - adobe
  - acrobat
  - registry
  - fix
date: 2026-05-11
source: "[[11-05-26]]"
publish: true
---
# Adobe Acrobat Reader Mode Fix

Fixed the issue where Acrobat DC would close (instead of reverting to Reader) if you didn't sign in with an Adobe licence after having had one previously.

## Fix

Set `bIsSCReducedModeEnforcedEx` to `1` in the Windows registry:

```
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Adobe\Adobe Acrobat\DC\FeatureLockDown
```

Value name: `bIsSCReducedModeEnforcedEx`
Value type: DWORD
Value data: `1`

→ [[set-adobe-reader-mode]] (batch script to apply the key)

## Notes

- LEAP issue (not downloading to cache / not opening) may be a separate, unrelated problem — unconfirmed as of 11 May.
