---
category: reference
tags:
  - outlook
  - myob
  - antivirus
  - programmaticaccess
publish: true
---
## Symptom

Outlook shows a popup: a program is trying to access email address information stored in Outlook, with an option to allow access for a set time. Hitting Allow doesn't stick — MYOB freezes and Outlook/MYOB stop talking to each other. Windows Defender also shows a warning that antivirus is off, even though it isn't.

Seen more than once (e.g. Leanne at Nepean) — usually tied to antivirus/security software interacting with Outlook's Programmatic Access (the same mechanism that guards against malware silently reading your inbox, but also gates legitimate integrations like MYOB).

## Fix

1. Launch Outlook **as administrator** — the Programmatic Access settings can't be changed otherwise.
2. Go to **Trust Center > Programmatic Access**.
3. Adjust/disable the warning there.

## Notes

- Antivirus status may still report "Invalid" or show as off in Windows even after this is resolved — that's a red herring, not a sign the fix didn't work.
- If disabling the prompt doesn't stick, check the AV product's own Outlook add-in/protection settings, since some AVs re-trigger this on their own schedule.
