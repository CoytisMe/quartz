---
category: reference
tags:
  - power-automate
  - onedrive
  - fteam
  - notifications
date: 2026-05-21
publish: true
---
# Power Automate — fteam Change Notification (Rate Limited)

Sends a mobile notification when theFteam OneDrive folder is modified, with a max of one notification per minute.

Rate limiting works by reading/writing a timestamp to a text file in OneDrive and skipping the notification if one was sent less than 60 seconds ago.

**Timestamp file:** `OneDrive - Coytis/Claude/fteam_last_notified.txt`

---

## Flow Steps

### Trigger
- **When a file is created or modified (OneDrive)**
- Folder: path to your theFteam folder

---

### Action 1 — Get file content
- **Action:** Get file content (OneDrive)
- **File:** `/Claude/fteam_last_notified.txt`

---

### Action 2 — Parse the timestamp
- **Action:** Initialize variable
- **Name:** `lastNotified`
- **Type:** String
- **Value:** `@{body('Get_file_content')}`

---

### Action 3 — Check if 60 seconds have passed
- **Action:** Condition
- **Expression (left):** `@{div(sub(ticks(utcNow()), ticks(variables('lastNotified'))), 10000000)}`
- **Operator:** is greater than
- **Value:** `60`

---

### If YES branch (more than 60 seconds ago)

#### Action 4 — Send notification
- **Action:** Send me a mobile notification (Power Automate app)
- **Text:** `theFteam folder was updated`

#### Action 5 — Update timestamp file
- **Action:** Update file (OneDrive)
- **File:** `/Claude/fteam_last_notified.txt`
- **Content:** `@{utcNow()}`

---

### If NO branch
- Leave empty — flow just ends with no notification

---

## Notes
- The `ticks()` expression converts a datetime to 100-nanosecond intervals — dividing the difference by 10,000,000 gives seconds
- The txt file starts with a dummy date (`2000-01-01T00:00:00`) so the first trigger always fires
- If the file read fails for any reason, add a "Configure run after" on Action 2 to handle errors gracefully
