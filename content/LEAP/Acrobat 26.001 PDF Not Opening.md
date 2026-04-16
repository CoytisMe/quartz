---
category: how-to
publish: true
tags:
  - leap
  - adobe
  - acrobat
  - troubleshooting
date: 2026-04-16
---

# LEAP – Acrobat 26.001 PDF Not Opening

Investigated 2026-04-16. Affecting multiple clients simultaneously — root cause is Adobe's emergency security patch (26.001.21431, released April 14 2026) breaking the LEAP-Acrobat integration.

---

## Symptoms

- PDF downloads to cache (`AppData\Local\LEAP Desktop\CDE\...\AcrobatDocuments\`) correctly and at correct file size
- Adobe Acrobat opens but shows home screen / recent documents — does not load the document
- Trying to open again from LEAP gives "document already open" error
- Cache folder accumulates duplicate numbered copies (`Cyber Security Brochure`, `_1`, `_2`... `_102`) — all same size, same timestamp — because LEAP creates a new copy each attempt rather than reusing the existing file
- Wiping and recreating the cache folder does not fix the underlying issue

## Root Cause

Adobe Acrobat version **26.001.21431** (emergency security patch for CVE-2026-34621, released April 14 2026) broke the COM/DDE handoff that LEAP uses to tell Acrobat to open a specific file. Acrobat launches successfully but silently drops the file open command.

LEAP's "already open" detection sees Acrobat running and assumes success — so it won't retry. The actual document never loads.

**Do not roll back Acrobat** — 26.001.21411 and 26.001.21431 are emergency patches for an actively exploited zero-day. Rolling back reintroduces a real security risk.

---

## What Was Tried (and Didn't Work)

- Wiping the AcrobatDocuments cache folder and recreating — files land fine, issue persists
- Adobe Acrobat Repair (via appwiz.cpl)
- Running `InstallLauncher.exe` from `C:\ProgramData\LEAP Office\Cloud\Extras\Acrobat Extras\`
- Disabling New Acrobat interface (File → Disable New Acrobat Reader)
- Disabling Protected Mode / Enhanced Security (Edit → Preferences → Security Enhanced)

---

## Steps Worth Trying

These are the remaining options from LEAP's own support documentation for this symptom.

### 1. Run LEAPForAcrobatSetup.exe specifically
Different to InstallLauncher.exe — this is the actual add-in installer.

```
C:\ProgramData\LEAP Office\Cloud\Extras\Acrobat Extras\LEAPForAcrobatSetup.exe
```

Run with Acrobat fully closed.

### 2. Check for LeapCommunicator.api in Acrobat plugins folder
This is the add-in file LEAP uses to communicate with Acrobat. If missing or corrupt, the handoff silently fails.

**Acrobat Reader:**
```
C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\plug_ins\
```

**Acrobat Pro:**
```
C:\Program Files\Adobe\Acrobat DC\Acrobat\plug_ins\
```

Look for `LeapCommunicator.api`. If missing — copy from a working machine or reinstall the add-in.

### 3. Verify LEAP add-in is visible in Acrobat
Open Acrobat → Tools. LEAP should appear as an add-in. If it's not there, the plugin isn't registering.

### 4. Check Acrobat is set as default PDF handler
LEAP does not support alternative PDF handlers (Edge PDF, etc). Confirm via Windows Settings → Default Apps → .pdf → Adobe Acrobat.

---

## LEAP Support Notes

LEAP support is historically slow on Acrobat integration issues. This is a known recurring pattern — every major Acrobat update cycle tends to break the integration until LEAP pushes a patch.

When logging with LEAP support, reference:
- Adobe Acrobat version: **26.001.21431**
- Symptom: Acrobat opens to home screen, does not load document from matter
- Already tried: Adobe repair, InstallLauncher, Adobe security settings
- Affects: multiple clients/machines simultaneously

LEAP community article for this specific symptom (requires login):
- https://community.leap.com.au/s/article/Adobe-Acrobat-Launches-but-Fails-to-Load-a-PDF-Document-from-a-LEAP-Matter

LEAP AU general PDF issues:
- https://community.leap.com.au/s/article/Issues-Opening-PDFs

---

## Sources

- [Adobe CVE-2026-34621 / Acrobat 26.001.21431 – Malwarebytes](https://www.malwarebytes.com/blog/news/2026/04/simply-opening-a-pdf-could-trigger-this-adobe-reader-zero-day)
- [LEAP Community – Unable to Open PDFs from a Matter (US)](https://community.leap.us/s/article/Issues-Opening-PDFs-US)
- [LEAP Community – Acrobat Launches but Fails to Load PDF](https://community.leap.com.au/s/article/Adobe-Acrobat-Launches-but-Fails-to-Load-a-PDF-Document-from-a-LEAP-Matter)
- [Adobe Community – LEAP integration not working (historical thread)](https://community.adobe.com/t5/acrobat-discussions/integration-with-leap-not-working/td-p/10937684)
