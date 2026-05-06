---
category: work-note
tags:
  - permissions
  - powershell
  - windows
  - ntfs
  - icacls
  - leap
date: 2026-04-24
publish: true
---
## Fuuucking LEAP didn't have permissions to it's work shit code
Pretty sure this can be done by right clicking but hey it's fun to make Claude write stuff

## His Notes

## The Problem

Windows NTFS permissions work via inheritance — child items (files and subfolders) normally inherit permissions from their parent folder automatically. Sometimes this breaks:

- Someone has manually set explicit permissions on a file or folder, which overrides inherited ones
- A file was copied from somewhere else and brought its own permissions with it
- An app or installer set its own permissions on an exe or folder during setup

The result is a parent folder with the right permissions, but a child item that ignores them.

---

## The Fix — Single Item

### icacls /reset (simplest)
```powershell
icacls "C:\Path\To\File.exe" /reset
```
Strips all explicit permissions from the item and restores inheritance from the parent. The item will then pick up whatever the parent folder has. Best for a single file or folder where you just want it to behave normally.

### PowerShell — copy parent ACL to child
```powershell
$acl = Get-Acl "C:\Path\To\Folder"
$acl.SetAccessRuleProtection($false, $true)
Set-Acl -Path "C:\Path\To\Item" -AclObject $acl
```
- `Get-Acl` reads the permission set (ACL = Access Control List) from the parent folder
- `SetAccessRuleProtection($false, $true)` — first param `$false` means "don't block inheritance", second param `$true` means "keep existing inherited rules"
- `Set-Acl` applies that ACL to the target item

More explicit than `/reset` — useful if you want to copy a specific folder's permissions rather than just restoring inheritance.

---

## The Fix — Whole Folder Tree

See `fixpermissions.ps1` in Scripts folder.

Prompts for a folder path, reads its permissions, then applies them to every file and subfolder underneath it — resetting any explicit permissions that were blocking inheritance.

---

## Key Concepts

**ACL (Access Control List)** — the full set of permissions attached to a file or folder. Made up of ACEs (Access Control Entries), one per user or group.

**Inheritance** — child items automatically get a copy of the parent's ACL. When you change a parent's permissions you get the option to push that down to all children — that's what the "Replace child object permissions" button in the GUI does.

**Explicit vs Inherited permissions** — inherited permissions come from the parent. Explicit permissions are set directly on the item and override inherited ones. `icacls /reset` removes explicit permissions, leaving only inherited.

**icacls** — built-in Windows command line tool for managing file/folder permissions. Faster than PowerShell for simple jobs, available on every Windows machine without any modules.
