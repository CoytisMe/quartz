---
category: how-to
date: 2026-09-26
publish: true
tags:
  - adobe
  - acrobat
  - reinstall
  - uninstall
  - troubleshooting
---
# Adobe Acrobat Clean Reinstall

A normal uninstall/reinstall leaves junk behind, so do it properly. It's also worth doing when someone has "fixed" Acrobat by just turning off its Outlook add-in: that hides the problem, and the add-in tends to come back.

**Allow about 30 minutes.**

## Steps

1. **Uninstall Acrobat normally** via `appwiz.cpl`
2. **Download and run the Acrobat Cleaner Tool** (AcroCleaner) from Adobe to strip out what the uninstaller missed
3. **Delete the leftover folders** if they're still there:
   - `C:\Program Files\Adobe`
   - `C:\Program Files\Common Files\Adobe`
4. **Reboot**
5. **Download and install Acrobat using the standalone installer**
   - **NEVER use Creative Cloud** to install it
6. **Sign the user's Adobe account back in**, or get them to do it

## If it was an Outlook problem

If the original complaint was Outlook freezing or crawling, check the **other Outlook add-ins before blaming Adobe**. Practice-management add-ins (LEAP in particular) syncing huge contact sets can freeze Outlook on send and look exactly like an Adobe problem. A clean Acrobat reinstall won't hurt, but it might not be the fix.

If the Acrobat add-in gets disabled rather than removed, remove it properly by running Outlook as admin, then File → Options → Add-ins → COM Add-ins.

also... [[Adobe Kill Command]], [[LEAP Adobe Add-in Script]], [[LEAP Full Uninstall]]
