---
category: reference
tags:
  - sharepoint
  - network-drive
  - webdav
  - how-to
date: 2026-05-12
publish: true
---
### Want Sharepoint? (Correct Opinion)
### Hate OneDrive (Correct Opinion)

See below

Uses WebDAV protocol to mount a SharePoint document library as a mapped drive in File Explorer. No OneDrive sync client required.

> **Note:** Microsoft recommends the OneDrive sync client over this method for performance and consistency. This approach has limitations — no offline access, version history may be lost, metadata/alerts won't work. Use when OneDrive isn't an option.

---

## Prerequisites

- Windows 10 or 11
- WebClient service running
- Microsoft 365 account with access to the library
- SharePoint site **URL**

---

## Step 1 — Enable WebClient Service

```powershell
Get-Service webclient
Set-Service webclient -StartupType Automatic -PassThru
Start-Service webclient
```

---

## Step 2 — Authenticate via Edge (IE Mode)

The WebDAV connection needs a Modern Auth token. Do this once — token lasts 90 days and auto-renews while the session is active.

1. Open Edge → Settings → Default browser → Internet Explorer mode pages → **Add** your SharePoint URL (e.g. `https://tenant.sharepoint.com`)
2. Browse to `https://login.microsoftonline.com/` in IE Mode
3. Sign in with the M365 account and tick **Stay signed in**

---

## Step 3 — Add SharePoint to Trusted Sites

1. Open **Internet Options** (run `inetcpl.cpl`)
2. Security tab → Trusted Sites → **Sites**
3. Add the SharePoint URL: `https://tenant.sharepoint.com`
4. Click **Custom Level** → User Authentication → Logon → set to **Automatically logon with current user name and password**

---

## Step 4 — Map the Drive

**Via File Explorer:**
1. This PC → Map Network Drive (top ribbon or right-click)
2. Choose a drive letter
3. In the Folder field enter the library URL — e.g:
   `https://tenant.sharepoint.com/sites/sitename/Shared Documents`
4. Tick **Reconnect at sign-in**
5. Click Finish and enter M365 credentials if prompted

**Via Command Line:**
```cmd
net use Z: "https://tenant.sharepoint.com/sites/sitename/Shared Documents" /persistent:yes
```

**Via PowerShell:**
```powershell
New-PSDrive -Name "Z" -Root "https://tenant.sharepoint.com/sites/sitename/Shared Documents" -PSProvider FileSystem -Persist
```

---

## Common Errors

**Access Denied on mapping:**
Site not in Trusted Sites or auth token expired. Re-authenticate via IE Mode in Edge and check Trusted Sites.

**0x80070043 — Network name cannot be found:**
WebClient service not running. Run the Step 1 commands.

**Drive doesn't reconnect after reboot:**
Re-authenticate via IE Mode. Token may have expired (90 day limit).

---

## Limitations

- No offline access — requires internet connection
- 5,000 item limit per library view
- Large file transfers are slower than OneDrive sync
- Metadata columns, alerts, and version history may not work as expected
- Token expires every 90 days — requires re-authentication in IE Mode
