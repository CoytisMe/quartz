---
category: troubleshooting
date: 2026-07-02
publish: true
tags:
  - outlook
  - public-folders
  - permissions
  - calendar
  - exchange
  - powershell
---
## What a Public Folder calendar actually is

Not a mailbox, not a shared mailbox — lives in the org's Public Folder mailbox hierarchy, a completely different object type from `Get-Mailbox`. Common legacy pattern for "shared calendars" that predates modern shared mailboxes, still everywhere in older tenants.

**Tell-tale signs it's a Public Folder and not a shared mailbox calendar:**
- `Get-Mailbox -RecipientTypeDetails SharedMailbox | Where-Object {$_.DisplayName -like "*name*"}` returns nothing.
- On a user who already has it working, the folder path shows as `\\Public Folders - <UPN>\Favourites`, type "Folder containing calendar items (MAPI)".
- `Add-MailboxFolderPermission` / OWA "Add calendar > From directory" does nothing — wrong cmdlet entirely for Public Folders.
- Outlook's native Share Calendar button does nothing useful either, doesn't apply to Public Folders.
- Multiple people show as "Owner" — normal for legacy public folders, don't mistake this for M365 Group ownership.

## Finding it

```powershell
Get-PublicFolder -Identity "\" -Recurse | Where-Object {$_.Name -like "*name*"}
```
Returns Name + Parent Path. **The folder is very likely nested** (e.g. under a container like `FCAS Public Folders`) — the full identity for every other command is `\Parent\FolderName`, not just `\FolderName`. Using the leaf name alone throws `Couldn't find public folder`.

## Checking / granting permissions

```powershell
# check current permissions (use the FULL path incl. parent)
Get-PublicFolderClientPermission -Identity "\Parent Folder\"

# grant access
Add-PublicFolderClientPermission -Identity "\Parent Folder\Westgate? Y/N Tracking" -User user@domain.com -AccessRights PublishingEditor

# if the user already has an entry, Add- errors out with "couldn't find" or already-exists - use Set- instead
Set-PublicFolderClientPermission -Identity "\Parent Folder\Westgate? Y/N Tracking" -User user@domain.com -AccessRights PublishingEditor

# clean up if you fat-fingered the parent container instead of the actual calendar folder
Remove-PublicFolderClientPermission -Identity "\Parent Folder\Westgate? Y/N Tracking" -User user@domain.com -AccessRights PublishingEditor
```

Permissions **do not cascade** from parent to child - granting on the container folder does nothing for the calendar folder inside it. Easy mistake: `Get-PublicFolder -Recurse` shows the calendar's parent path, and it's tempting to just grant on that parent because the child identity string throws an error the first time (usually because you dropped the parent segment).

## Getting it to actually show up in Outlook

Permission alone never surfaces it. The user has to manually favourite the folder:

1. Classic Outlook desktop only. New Outlook / OWA don't reliably support browsing Public Folders.
2. Switch the nav pane to **Folder List** view - `Ctrl+6`, or the "..." at the bottom-left of the icon strip → **Folders**. This is different from the normal Folder Pane sidebar (Mail/Calendar view) - Folder List shows every folder in the profile including Public Folders.
3. Expand **Public Folders - `<UPN>`** → **All Public Folders** → the container → the actual calendar folder.
4. Right-click it → **Add to Favorites** (or drag it onto the Favorites folder shown just below).
5. Switch to Calendar view - it now shows under **Other Calendars**.

Once favourited on one device it seems to autopopulate into OWA and other devices on the same account after a while. If it hasn't shown up elsewhere within a day, repeat the Favorites step manually on that device.
