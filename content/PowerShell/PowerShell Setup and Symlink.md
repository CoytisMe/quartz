---
category: work-note
tags:
  - powershell
  - setup
  - reference
date: 2026-04-27
publish: true
---
# PowerShell Environment Setup

Everything needed to get a fresh terminal up to speed with the scripts folder.

---

## 1. Scripts Folder Symlink

Point the local Scripts folder at the OneDrive source so all scripts are available:

```powershell
New-Item -ItemType SymbolicLink -Path "C:\Users\Rick\Scripts" -Target "C:\Users\Rick\OneDrive - Coytis\Scripts"
```

> Requires admin or Developer Mode enabled.

More on [[Symlinks]]

---

## 2. Set Execution Policy

Allow local scripts to run:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

## 3. Install Modules

### Exchange Online
```powershell
Install-Module ExchangeOnlineManagement
```

### Microsoft Graph
```powershell
Install-Module Microsoft.Graph.Identity.SignIns
Install-Module Microsoft.Graph.Users
```

---

## 4. Verify

```powershell
Get-Module ExchangeOnlineManagement -ListAvailable
Get-Module Microsoft.Graph.Identity.SignIns -ListAvailable
Get-Module Microsoft.Graph.Users -ListAvailable
```

---

## Notes

- Scripts live at `C:\Users\Rick\OneDrive - Coytis\Claude\Scripts` — symlinked from `C:\Users\Rick\Scripts`
- Exchange connects via browser (Chrome preferred for 1Password extension)
- Graph connects via WAM popup — `-ContextScope Process` on all scripts keeps sessions isolated per tab
