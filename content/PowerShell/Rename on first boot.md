---
category: how-to
date: 2026-06-06
publish: true
tags:
  - powershell
  - proxmox
  - windows
  - template
---

## Rename on First Boot

Used for linked template VMs via Proxmox.

On boot it pops up, asks for new name, then asks to reboot.

**Needs to be set up on the template**

 Save this script to C:\Scripts\rename-on-clone.ps1 (create the folder if it doesn't exist):

  ```powershell
  if ($env:COMPUTERNAME -eq "win11temp") {
      $newName = Read-Host "Enter a name for this machine"
      if ($newName) {
          Unregister-ScheduledTask -TaskName "RenameOnClone" -Confirm:$false
          Rename-Computer -NewName $newName -Force
          $restart = Read-Host "Restart now? (y/n)"
          if ($restart -eq 'y') { Restart-Computer -Force }
      }
  }
  ```

  Then register the scheduled task:

  ```powershell
  $action  = New-ScheduledTaskAction -Execute "powershell.exe" `
                 -Argument '-NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\rename-on-clone.ps1"'
  $trigger = New-ScheduledTaskTrigger -AtLogOn
  Register-ScheduledTask -TaskName "RenameOnClone" -Action $action -Trigger $trigger -RunLevel Highest -Force
  ```

Flow once it's templated: 
Spin up clone → log in → PowerShell window pops up → type a name → machine renames and reboots → task deletes itself → never prompts again.
