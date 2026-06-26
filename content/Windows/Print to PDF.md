---
category: tech
tags:
  - printing
  - windows
  - troubleshooting
date: 2026-06-24
publish: true
source: "[[24-06-26]]"
---
## Print to PDF not working?
lol how? isn't that just built in? 

Try setting up a virtual PDF printer via Windows:

- Add Printer
- Local or Network
- File: (Print to File)
- "Print to PDF" driver should be under Microsoft in the manufacturer list

### Also Try
- "Add or remove Windows Features" - Turn off, reboot, turn on, reboot.
- Find and uninstall in device manager
- Reinstall Windows

Have actually seen any of this work cept the last one, good luck everybody else.

## Followup

Microsoft's built-in "Microsoft Print to PDF" driver usually just works out of the box — if it's not showing under Manufacturers, the feature itself may be disabled. Worth checking: Settings → Apps → Optional Features → "Microsoft Print to PDF" is installed; and that the Print Spooler service is running. If it's still missing after that, a repair install of the driver via `pnputil` or re-enabling the optional feature usually fixes it.
