---
category: reference
date: 2026-06-18
publish: true
tags:
  - server
  - windows
  - proxmox
  - template
  - vm
---

# coytishost — Windows VMs

Two Windows 11 VMs: a general-purpose daily driver and a linked-clone template for spinning up disposable client-work VMs.

---

## WinVMGeneral (VM 100)

General-purpose Windows 11 VM. Currently hosting a client cloud backup job (being decommissioned). Access via RDP.

The plan is to use the below template for the backup and bs jobs in the future but this one is in prog and will be for a year because some clients give many less of fuck than me. After that it'll mostly be for streaming my Archery camera to discord and allowing my to have more incognito windows at once.

| | |
|---|---|
| **OS** | Windows 11 |
| **Storage** | NVMe (OS) + 2TB HDD |
| **Access** | RDP |

---

## Win11 Linked Clone Template (VM 104)

These things are fucking sick, make a windows, crank it hard with whatever bs you want, delete it.

Easy convenient, makes you feel powerful.

A clean Windows 11 base image for spinning up disposable VMs — client work, OneDrive syncs, Outlook backups, troubleshooting sessions. Linked clones share the template's base disk and only write their own changes on top. Creating one takes seconds; deleting it leaves the template untouched.

Requires LVM-thin storage (default Proxmox install) for snapshot support.

| | |
|---|---|
| **OS** | Windows 11 |
| **VMID** | 104 |
| **Name** | `win11template` |
| **Storage** | `local-lvm` (NVMe, LVM-thin) |
| **TPM** | Hardware TPM 2.0 |

---

### Template Setup (one-time)

I have all these set as bookmarks in my Command Pallet, again it's almost about feeling powerful.

1. Shut down the daily driver VM and detach the HDD in Proxmox Hardware tab (don't delete it)
2. Full-clone the daily driver → name it `win11template`
3. Reattach the HDD to the original VM and start it back up
4. On the clone, shrink C: — run this first in an admin PowerShell to free up space:

```powershell
powercfg /h off
Disable-ComputerRestore -Drive "C:\"
vssadmin delete shadows /all /quiet
Disable-BitLocker -MountPoint "C:"
# Wait for FullyDecrypted before shrinking:
Get-BitLockerVolume -MountPoint "C:" | Select-Object VolumeStatus, EncryptionPercentage
```

If Disk Management won't shrink past unmovable files, boot a GParted Live ISO instead (disable Secure Boot in VM UEFI first).

5. Rename the template machine to `WIN11-TEMPLATE`:

```powershell
Rename-Computer -NewName "WIN11-TEMPLATE" -Force -Restart
```

6. Drop this script at `C:\Scripts\rename-on-clone.ps1` on the template:

```powershell
if ($env:COMPUTERNAME -eq "WIN11-TEMPLATE") {
    $newName = Read-Host "Enter a name for this machine"
    if ($newName) {
        Unregister-ScheduledTask -TaskName "RenameOnClone" -Confirm:$false
        Rename-Computer -NewName $newName -Force
        $restart = Read-Host "Restart now? (y/n)"
        if ($restart -eq 'y') { Restart-Computer -Force }
    }
}
```

7. Register a scheduled task to run it at logon:

```powershell
$action  = New-ScheduledTaskAction -Execute "powershell.exe" `
               -Argument '-NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\rename-on-clone.ps1"'
$trigger = New-ScheduledTaskTrigger -AtLogOn
Register-ScheduledTask -TaskName "RenameOnClone" -Action $action -Trigger $trigger -RunLevel Highest -Force
```

8. Shut down the clone → right-click in Proxmox → **Convert to Template**

---

### Day-to-Day: New and Delete Scripts

Scripts live at `C:\Users\Rick\Scripts\` and are launched via PowerToys Run.

**new-client-vm.ps1** — prompts for a name, picks the next free VMID, clones template 104, starts it:

```powershell
$name = Read-Host "VM name"

Write-Host "`nConnecting to host..."
ssh -o StrictHostKeyChecking=no root@[HOST_IP] "NEWID=`$(pvesh get /cluster/nextid) && qm clone 104 `$NEWID --name $name --full 0 && qm start `$NEWID && echo `$NEWID"
Write-Host "`nDone - $name is starting"
```

**delete-client-vm.ps1** — lists linked clones of template 104, pick by number to delete:

```powershell
Write-Host "Fetching linked clones of template 104..."

$bash = @'
for vmid in $(qm list | awk 'NR>1 {print $1}'); do
  if lvs --noheadings -o name,origin 2>/dev/null | grep -q "vm-$vmid.*base-104"; then
    name=$(qm config $vmid | grep "^name:" | awk '{print $2}')
    echo "$vmid $name"
  fi
done
'@

$encoded = [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes($bash))
$raw = ssh -o StrictHostKeyChecking=no root@[HOST_IP] "echo $encoded | base64 -d | bash"

if (-not $raw) { Write-Host "No linked clones found."; exit }

$vms = @($raw) | Where-Object { $_ -match '^\d+\s' } | ForEach-Object {
    $parts = $_.Trim() -split '\s+', 2
    [PSCustomObject]@{ Id = $parts[0]; Name = $parts[1] }
}

for ($i = 0; $i -lt $vms.Count; $i++) {
    Write-Host "  [$($i + 1)] $($vms[$i].Name)  (VM $($vms[$i].Id))"
}

$choice = Read-Host "Select VM to delete"
$idx    = [int]$choice - 1
$target  = $vms[$idx]
$confirm = Read-Host "Delete '$($target.Name)' (VM $($target.Id))? (y/n)"
if ($confirm -ne 'y') { Write-Host "Cancelled."; exit }

ssh -o StrictHostKeyChecking=no root@[HOST_IP] "qm stop $($target.Id) --skiplock 2>/dev/null; qm destroy $($target.Id) --purge"
Write-Host "Deleted."
```

---

### Modifying the Template

To make changes, untemplate it from the Proxmox shell, edit, then re-template:

```bash
# Untemplate
qm set 104 --template 0

# Unlock and rename base disks
lvchange -prw /dev/pve/base-104-disk-0
lvrename pve/base-104-disk-0 pve/vm-104-disk-0
lvchange -prw /dev/pve/base-104-disk-1
lvrename pve/base-104-disk-1 pve/vm-104-disk-1

# Patch config to match
sed -i 's/base-104-disk/vm-104-disk/g' /etc/pve/qemu-server/104.conf

# If TPM fails on start, reset it
qm set 104 --delete tpmstate0
qm set 104 --tpmstate0 local-lvm:4,version=v2.0

qm start 104
```

After making changes:

```bash
qm shutdown 104 --timeout 60 && qm template 104
```

---

### Notes

- Install tools on the template before converting — every clone inherits them
- Don't log into personal Microsoft accounts on the template
- Every fresh clone has the same machine name and SID — fine since nothing is domain-joined
- The rename scheduled task removes itself after first run
