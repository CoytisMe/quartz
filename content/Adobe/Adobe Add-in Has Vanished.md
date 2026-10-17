---
category: tech
date: 2026-10-17
publish: true
tags:
  - kbdraft
  - publickb
  - adobe
  - acrobat
  - outlook
  - leap
  - addin
---
# Adobe Add-in Has Vanished

Checklist for an Adobe add-in or plugin that's gone missing or stopped working, in Outlook or in Acrobat. Three cases in about two weeks, with different mechanisms but the same "it's gone" symptom.

Related: [[Adobe Acrobat Clean Reinstall]], [[LEAP Adobe Add-in Script]], [[Acrobat 26.001 PDF Not Opening]], [[Print to PDF]]

## 1. Outlook: "Save as Adobe PDF" missing (PDFMaker)
- Cause: the Adobe PDFMaker COM add-in crashed and **Outlook disabled it**.
- Ticking it back on in Options > Add-ins **won't stick**.
- Fix: **File > Manage COM Add-ins** (the "slow and disabled add-ins" button) > **Disabled Items** > re-enable it.
- If it keeps disabling itself, Outlook is killing it for slow loading. Set **"Always enable this add-in"**, or repair Acrobat.
- Note: Print to PDF isn't the same thing. Users notice the difference.

## 2. Acrobat: third-party plugin (e.g. LEAP) not working, PDFs won't open from the app
- Acrobat > Edit > Preferences > **Plugins**.
- **The list is inverted: ticked = disabled.** The menu says "Uncheck a plugin to enable it".
- A plugin that *looks* ticked/on is actually off. Untick it and restart Acrobat.
- Check this **before** running the add-in reset script or reinstalling.

## 3. Outlook add-in turned off by someone else
- An earlier "fix" can be someone switching the add-in off. It comes back, or it leaves the real problem in place.
- If an add-in needs removing properly, run Outlook as admin and remove it there.
- If Acrobat itself is the problem, do a full clean reinstall ([[Adobe Acrobat Clean Reinstall]]).
- In the case seen, Adobe turned out to be a red herring. The Outlook slowness was the LEAP add-in syncing ~22k contacts.

## Quick order of checks
1. Outlook > File > Manage COM Add-ins > Disabled Items
2. Acrobat > Preferences > Plugins (remember the list is inverted)
3. Restart the app. After a LEAP update, a restart alone brought the LEAP button back.
4. Run the add-in reset script
5. Clean reinstall
