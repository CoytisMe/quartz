---
category: how-to
tags:
  - sharepoint
  - onedrive
  - powershell
  - ninjaone
  - file-explorer
  - sync
date: 2026-04-01
publish: true
---
# SharePoint - Sync Library to File Explorer

## Overview

Two methods to get a SharePoint document library showing in a user's File Explorer as a mapped drive equivalent. Neither requires connecting to the machine — one is user-self-serve via email link, the other can be pushed via NinjaOne.

---

## Method 1 — Direct Sync Link (Email to User)

SharePoint generates a sync URL directly from the library. Send this to the user — they click it, OneDrive opens and prompts them to confirm sync. They just need to click **Sync** and sign in if asked.

### How to get the URL

1. Go to the SharePoint document library in a browser
2. Click **Sync** in the top toolbar
3. The browser will prompt to open OneDrive — instead, copy the URL from the address bar just before it opens, or use browser dev tools to capture the `odopen://` link
4. Alternatively: in the library, click **⋯ > Details** then grab the URL and construct it manually (see below)

### Easier method to get the odopen:// link

1. Open the SharePoint library
2. Click **Sync** — let it open on *your* machine
3. In File Explorer, right-click the synced folder > **Settings** > note the site/library info
4. Or just send users the direct SharePoint library URL and tell them to click Sync themselves — SharePoint handles the `odopen://` handoff automatically

### What to tell the user in the email

> Click the link below, then click **Sync** when prompted. If it asks you to sign in, use your work email and password. Once done, you'll find the company drive in File Explorer under [Organisation Name].

---

## Method 2 — NinjaOne Script (Push to Entra-Joined or Work/School Machines)

### Requirements
- Must run **as the logged-on user**, not SYSTEM
- OneDrive must already be installed and signed in with their work account
- Works best on Entra-joined machines; may still prompt for sign-in on local account + Work/School setups

### PowerShell Command

```powershell
Start-Process "odopen://sync?siteId=<siteId>&webId=<webId>&listId=<listId>&userEmail=<userEmail>&webtitle=<SiteName>&listtitle=<LibraryName>"
```

### How to get the IDs

Run this in SharePoint Online PowerShell (or Graph Explorer):

```powershell
Connect-SPOService -Url https://<tenant>-admin.sharepoint.com

$site = Get-SPOSite -Identity https://<tenant>.sharepoint.com/sites/<sitename>
$site.Id  # = siteId
```

For `webId` and `listId`, use PnP PowerShell:

```powershell
Connect-PnPOnline -Url https://<tenant>.sharepoint.com/sites/<sitename> -Interactive

$web = Get-PnPWeb
$web.Id  # = webId

$list = Get-PnPList -Identity "Documents"
$list.Id  # = listId
```

### NinjaOne Deployment Notes
- In NinjaOne, set the script to run as **Logged-on User** (not SYSTEM)
- The `odopen://` URI is handled by OneDrive — it will open and prompt the user to confirm sync
- If OneDrive isn't signed in, the user will be prompted — that's fine, just tell them to sign in with their work account and click Sync

---

## Notes

- Sync creates a folder in File Explorer separate from the user's personal OneDrive — this is intentional and preferred over shortcuts for the "company drive" feel
- Shortcuts (Add shortcut to OneDrive) are the newer MS-preferred method but merge into the user's OneDrive folder — not suitable here
- Silent/no-interaction sync is only possible on fully Entra-joined machines without MFA, which isn't the case for most clients
- For the mixed environment (some Entra-joined, some local + Work/School), email link is the most reliable universal method
