---
category: work-note
tags:
  - powershell
  - windows
  - disk
  - bitlocker
  - proxmox
date: 2026-05-29
---
# Windows Partition Shrink Prep

Commands to run before shrinking a Windows partition via Disk Management or GParted. Windows parks unmovable files (page file, hibernate, VSS snapshots, BitLocker metadata) mid-disk and blocks the shrink — these clear the blockers.

Run all in an **admin PowerShell**.

---

## 1 — Disable Hibernation

Not needed on VMs. Removes `hiberfil.sys`.

```powershell
powercfg /h off
```

---

## 2 — Disable System Restore & clear VSS snapshots

```powershell
Disable-ComputerRestore -Drive "C:\"
vssadmin delete shadows /all /quiet
```

---

## 3 — Disable the Page File

Must reboot after this one.

- Search → "Adjust the appearance and performance of Windows"
- Advanced → Virtual Memory → Change
- Uncheck "Automatically manage" → select C: → **No paging file** → Set → OK → Reboot

---

## 4 — Disable BitLocker (if enabled)

BitLocker encrypts the whole partition — nothing can move files until it's off. This is the most common reason the above steps still don't work.

```powershell
# Start decryption
Disable-BitLocker -MountPoint "C:"

# Check progress
Get-BitLockerVolume -MountPoint "C:" | Select-Object VolumeStatus, EncryptionPercentage
```

Wait for `FullyDecrypted` before attempting the shrink. Can take 20-40 mins depending on drive size.

---

## After Shrinking

Re-enable what you actually want back:

```powershell
# Re-enable page file — set back to "Automatically manage" in the same UI
# Re-enable System Restore if needed
Enable-ComputerRestore -Drive "C:\"
# Hibernation — only if needed (not on VMs)
powercfg /h on
```

Leave BitLocker off on VM templates — the Proxmox host controls disk access.
