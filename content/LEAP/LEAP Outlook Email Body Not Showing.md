---
category: work-note
date: 2026-07-01
publish: true
tags:
  - outlook
  - leap
  - troubleshooting
---

# LEAP Outlook — Email Body Not Showing (Only Attachments)

## Symptom

Emails opened from LEAP in Outlook were not displaying the email body — only the attachments were visible.

## Fix

Uninstalled the Adobe add-ins for Outlook. Email bodies came good immediately after.

## Why This Happens

Adobe's Outlook add-ins can interfere with LEAP's COM-based email integration, causing the message body pane to not render — leaving only the attachment list visible.

## If Adobe Add-ins Reinstall (They Will)

Adobe add-ins tend to reinstall themselves on Adobe or Office updates. To block them more permanently without needing to uninstall each time:

1. In Outlook: **File → Options → Add-ins**
2. At the bottom, set **Manage** to **COM Add-ins** → click **Go**
3. Uncheck any Adobe add-ins (e.g. *Acrobat PDFMaker Office COM Addin*)
4. Click **OK**

This marks them as disabled in Outlook's registry so even if Adobe drops the files back, Outlook won't load them automatically.

If they keep coming back after that, the nuclear option is via the registry:

```
HKEY_CURRENT_USER\SOFTWARE\Microsoft\Office\Outlook\Addins\<AddinName>
```

Set `LoadBehavior` to `0` — this overrides whatever the Adobe installer sets on next update.
