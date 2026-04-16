---
category: how-to
publish: true
tags:
  - dell
  - supportassist
  - disk-space
  - storage
  - snapshots
---
# SARemediation Snapshots

> [!tip] Safe to delete live
> Snapshots can be deleted while Windows is running — nothing will break. Only Dell's SupportAssist recovery is lost.

## How to Disable (and Delete Existing Snapshots)

1. Open **Dell SupportAssist** → gear icon (Options) → **Settings** → **System Repair** → toggle **OFF**
   - *Or:* Control Panel → System and Security → SupportAssist OS Recovery → Settings → System Repair → **Disable**
2. **Reboot** — existing snapshots will be purged automatically.

Leaving System Repair **OFF** permanently prevents new snapshots from being created.

## Notes

- Snapshots are stored at `C:\ProgramData\Dell\SARemediation\`
- A new snapshot is created ~30 min after each Windows startup when enabled
- Disabling removes Dell's built-in boot recovery — no other day-to-day impact
- The System Repair toggle only appears in SupportAssist if **Dell SupportAssist Remediation** is installed (check via Control Panel → Programs and Features)
