---
category: reference
tags:
  - powershell
  - windows
date: 2026-04-23
publish: true
---

## I'm note sure if these are in any way different to a shortcut

```powershell
New-Item -ItemType SymbolicLink -Path "C:\Users\Rick\Scripts" -Target "C:\Users\Rick\OneDrive - Coytis\Scripts"
```

Works through PowerShell, not through Quartz

## Symlinks vs Shortcuts — they are different

Shortcuts (`.lnk` files) are a Windows shell concept — they only work in File Explorer and the Start Menu. They don't exist at the filesystem level, so PowerShell, terminal, and most apps can't follow them.

Symlinks are filesystem-level — the OS treats the symlink path as if it actually is the target. PowerShell, CMD, apps, everything follows them transparently. That's why `C:\Users\Rick\Scripts` works in terminal even though it's physically living in OneDrive.

**Practical difference:**
- Shortcut: double-click in Explorer, useless in terminal
- Symlink: works everywhere, invisible to whatever's using it

You need admin or Developer Mode enabled to create symlinks on Windows — that's the main gotcha.

## Other symlink uses worth knowing

- Point `C:\Users\Rick\Documents` at an OneDrive folder to keep Documents synced without moving the folder
- Point app config folders at a OneDrive location for portable config across machines
- Useful anywhere an app hardcodes a path but you want the data somewhere else
