---
category: tech
tags:
  - ninja
  - rmm
  - uninstall
date: 2026-06-25
publish: true
---

# Im publishing this and the related note
The Claude follow up made me laugh

Check in `C:\Program Files (x86)\NinjaOne` for an uninstaller. If it's there, it worked when the Control Panel one didn't.

*Note: This agent hadn't checked in for 3 days and had been removed from the website — unclear if not checking in after removal affects which uninstall.exe works.*

Otherwise run:
```
"C:\Program Files (x86)\NinjaOne\NinjaRMMAgent.exe" -disableUninstallPrevention
```
Then run the Control Panel uninstall, or run:

## Followup

Related: [[Ninja - Create List of Devices]]. This entry trails off mid-thought — the command after "or run:" never got written down, so the full sequence isn't captured yet. Worth finishing the note next time this comes up, and maybe turning the disable-prevention + uninstall sequence into a one-liner in Scripts/ once the missing step is confirmed.
