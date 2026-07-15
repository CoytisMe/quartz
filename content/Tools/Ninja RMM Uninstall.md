---
category: tech
tags:
  - ninja
  - rmm
  - uninstall
date: 2026-06-25
publish: true
---

# Im publishing this and the related note
The Claude follow up made me laugh

Check in `C:\Program Files (x86)\NinjaOne` for an uninstaller. If it's there, it worked when the Control Panel one didn't.

*Note: This agent hadn't checked in for 3 days and had been removed from the website — unclear if not checking in after removal affects which uninstall.exe works.*

Otherwise run:
```
"C:\Program Files (x86)\NinjaOne\NinjaRMMAgent.exe" -disableUninstallPrevention
```
Then run the Control Panel uninstall, or run:

### Script that actually worked.

```powershell
<#
.SYNOPSIS
    Removes the NinjaOne (NinjaRMM) Agent from a Windows endpoint.

.DESCRIPTION
    Official-style NinjaOne agent removal script (based on NinjaOne's published
    "Remove an Endpoint Agent on Windows Using a Custom Script" guide), adapted
    for direct local execution. It:
      - Relaunches itself elevated if not already running as Administrator
      - Disables uninstall prevention on the agent
      - Runs the MSI uninstaller silently
      - Stops/removes related services and processes
      - Removes leftover install/data directories
      - Cleans up registry remnants (including Ninja Remote, if present)
    Logs everything to: C:\Windows\Temp\NinjaOneAgentRemoval.log

.NOTES
    Run this in an elevated PowerShell session (or just run the .ps1 file;
    it will prompt for elevation via UAC if needed).
#>

function Write-LogEntry {
    param (
        [Parameter(Mandatory = $true)]
        [string]$Message
    )

    $LogPath = "$env:windir\temp\NinjaOneAgentRemoval.log"
    $TimeStamp = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'
    Add-Content -Path $LogPath -Value "$TimeStamp - $Message"
    Write-Host "$TimeStamp - $Message"
}

function Uninstall-NinjaMSI {
    $Arguments = @(
        "/x$($UninstallString)"
        '/quiet'
        '/L*V'
        "$env:windir\temp\NinjaRMMAgent_uninstall.log"
        "WRAPPED_ARGUMENTS=`"--mode unattended`""
    )

    Start-Process "msiexec.exe" -ArgumentList $Arguments -Wait -NoNewWindow
    Write-LogEntry 'Finished running uninstaller. Continuing to clean up...'
    Start-Sleep 30
}

# Get current user context
$CurrentUser = New-Object Security.Principal.WindowsPrincipal $([Security.Principal.WindowsIdentity]::GetCurrent())
# Check that the user running the script is a member of the Administrator group
if (!($CurrentUser.IsInRole([Security.Principal.WindowsBuiltinRole]::Administrator))) {
    # UAC prompt will occur for the user to provide Administrator credentials and relaunch the session
    Write-LogEntry 'This script must be run with administrative privileges. Relaunching with elevation...'
    Start-Process powershell.exe "-NoProfile -ExecutionPolicy Bypass -File `"$PSCommandPath`"" -Verb RunAs
    Exit
}

$ErrorActionPreference = "SilentlyContinue"

Write-LogEntry 'Beginning NinjaRMM Agent removal...'
Write-LogEntry "Path to log file: $env:windir\temp\NinjaOneAgentRemoval.log"

$NinjaRegPath       = 'HKLM:\SOFTWARE\WOW6432Node\NinjaRMM LLC\NinjaRMMAgent'
$NinjaDataDirectory = "$($env:ProgramData)\NinjaRMMAgent"
$UninstallRegPath   = 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*'
$NinjaModulePath    = "$env:ProgramFiles\WindowsPowerShell\Modules\NJCliPSh"

if (!([System.Environment]::Is64BitOperatingSystem)) {
    $NinjaRegPath     = 'HKLM:\SOFTWARE\NinjaRMM LLC\NinjaRMMAgent'
    $UninstallRegPath = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*'
}

$NinjaInstallLocation = (Get-ItemPropertyValue $NinjaRegPath -Name Location -ErrorAction SilentlyContinue)
if ($NinjaInstallLocation) {
    $NinjaInstallLocation = $NinjaInstallLocation.Replace('/', '\')
}

if (!$NinjaInstallLocation -or !(Test-Path "$($NinjaInstallLocation)\NinjaRMMAgent.exe")) {
    $NinjaServicePath = ((Get-WmiObject Win32_Service | Where-Object { $_.Name -eq 'NinjaRMMAgent' }).PathName)
    if ($NinjaServicePath) { $NinjaServicePath = $NinjaServicePath.Trim('"') }
    if (!$NinjaServicePath -or !(Test-Path $NinjaServicePath)) {
        Write-LogEntry 'Unable to locate Ninja installation path. Continuing with cleanup...'
    }
    else {
        $NinjaInstallLocation = $NinjaServicePath | Split-Path
    }
}

if ($NinjaInstallLocation -and (Test-Path "$($NinjaInstallLocation)\NinjaRMMAgent.exe")) {
    Write-LogEntry 'Disabling uninstall prevention...'
    Start-Process "$($NinjaInstallLocation)\NinjaRMMAgent.exe" -ArgumentList "-disableUninstallPrevention NOUI" -Wait -NoNewWindow
}

$UninstallString = (Get-ItemProperty $UninstallRegPath | Where-Object {
    ($_.DisplayName -eq 'NinjaRMMAgent') -and ($_.UninstallString -match 'msiexec')
}).UninstallString

if (!($UninstallString)) {
    Write-LogEntry 'Unable to determine uninstall string. Continuing with cleanup...'
}
else {
    $UninstallString = $UninstallString.Split('X')[1].Trim()
    Write-LogEntry 'Running MSI uninstaller...'
    Uninstall-NinjaMSI
}

$NinjaServices = @('NinjaRMMAgent', 'nmsmanager', 'lockhart')
$Processes = @('NinjaRMMAgent', 'NinjaRMMAgentPatcher', 'njbar', 'NinjaRMMProxyProcess64')

foreach ($Process in $Processes) {
    $GetP = Get-Process $Process -ErrorAction SilentlyContinue
    if ($GetP) {
        try {
            Stop-Process $GetP -Force -ErrorAction Stop
            Write-LogEntry "Successfully stopped process: $($GetP.Name)"
        }
        catch {
            Write-LogEntry "Unable to stop $($GetP.Name): $($_.Exception.Message). Continuing..."
        }
    }
}

foreach ($NS in $NinjaServices) {
    if ($NS -eq 'lockhart' -and !(Test-Path "$NinjaInstallLocation\lockhart\bin\lockhart.exe")) {
        continue
    }
    if (Get-Service $NS -ErrorAction SilentlyContinue) {
        try {
            Write-LogEntry "Stopping service $($NS)..."
            Stop-Service $NS -Force -ErrorAction Stop
        }
        catch {
            Write-LogEntry "Unable to stop $($NS) service: $($_.Exception.Message)"
            Write-LogEntry 'Attempting to remove service...'
        }

        & sc.exe DELETE $NS | Out-Null
        Start-Sleep 5
        if (Get-Service $NS -ErrorAction SilentlyContinue) {
            Write-LogEntry "Failed to remove $($NS) service. Continuing with remaining removal steps..."
        }
        else {
            Write-LogEntry "Successfully removed $($NS) service."
        }
    }
}

if ($NinjaInstallLocation -and (Test-Path $NinjaInstallLocation)) {
    Write-LogEntry "Removing Ninja installation directory: $($NinjaInstallLocation)"
    try {
        Remove-Item $NinjaInstallLocation -Recurse -Force -ErrorAction Stop
        Write-LogEntry 'Successfully removed.'
    }
    catch {
        Write-LogEntry "Failed to remove Ninja installation directory: $($_.Exception.Message)"
        Write-LogEntry 'Continuing with removal attempt...'
    }
}

if (Test-Path $NinjaDataDirectory) {
    Write-LogEntry "Removing Ninja data directory: $($NinjaDataDirectory)"
    try {
        Remove-Item $NinjaDataDirectory -Recurse -Force -ErrorAction Stop
        Write-LogEntry 'Successfully removed.'
    }
    catch {
        Write-LogEntry "Failed to remove Ninja data directory: $($_.Exception.Message)"
        Write-LogEntry 'Continuing with removal attempt...'
    }
}

if (Test-Path $NinjaModulePath) {
    Write-LogEntry "Removing Ninja PowerShell module directory: $($NinjaModulePath)"
    try {
        Remove-Item $NinjaModulePath -Recurse -Force -ErrorAction Stop
        Write-LogEntry 'Successfully removed.'
    }
    catch {
        Write-LogEntry "Failed to remove Ninja PowerShell module directory: $($_.Exception.Message)"
        Write-LogEntry 'Continuing with removal attempt...'
    }
}

# --- Registry cleanup ---
$MSIWrapperReg       = 'HKLM:\SOFTWARE\WOW6432Node\EXEMSI.COM\MSI Wrapper\Installed'
$ProductInstallerReg = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18\Products'
$HKCRInstallerReg    = 'Registry::HKEY_CLASSES_ROOT\Installer\Products'

$RegKeysToRemove = [System.Collections.Generic.List[object]]::New()

(Get-ItemProperty $UninstallRegPath -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName -eq 'NinjaRMMAgent' }).PSPath | ForEach-Object { $RegKeysToRemove.Add($_) }
(Get-ItemProperty $ProductInstallerReg -ErrorAction SilentlyContinue | Where-Object { $_.ProductName -eq 'NinjaRMMAgent' }).PSPath | ForEach-Object { $RegKeysToRemove.Add($_) }
(Get-ChildItem $MSIWrapperReg -ErrorAction SilentlyContinue | Where-Object { $_.Name -match 'NinjaRMMAgent' }).PSPath | ForEach-Object { $RegKeysToRemove.Add($_) }
Get-ChildItem $HKCRInstallerReg -ErrorAction SilentlyContinue | ForEach-Object {
    if ((Get-ItemPropertyValue $_.PSPath -Name 'ProductName' -ErrorAction SilentlyContinue) -eq 'NinjaRMMAgent') {
        $RegKeysToRemove.Add($_.PSPath)
    }
}

$ProductInstallerKeys = Get-ChildItem $ProductInstallerReg -ErrorAction SilentlyContinue | Select-Object *
foreach ($Key in $ProductInstallerKeys) {
    $KeyName = $($Key.Name).Replace('HKEY_LOCAL_MACHINE', 'HKLM:') + "\InstallProperties"
    if (Get-ItemProperty $KeyName -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName -eq 'NinjaRMMAgent' }) {
        $RegKeysToRemove.Add($Key.PSPath)
    }
}

Write-LogEntry 'Removing registry items if found...'
if (($RegKeysToRemove | Measure-Object).Count -gt 0) {
    foreach ($RegKey in $RegKeysToRemove) {
        if (!([string]::IsNullOrWhiteSpace($RegKey))) {
            Write-LogEntry "Attempting to remove: $($RegKey)"
            try {
                Remove-Item $RegKey -Recurse -Force -ErrorAction Stop
                Write-LogEntry 'Successfully removed.'
            }
            catch {
                Write-LogEntry "Failed to remove registry key: $($_.Exception.Message)"
                Write-LogEntry 'Continuing with removal...'
            }
        }
    }
}

if (Test-Path $NinjaRegPath) {
    try {
        Write-LogEntry "Removing: $($NinjaRegPath)"
        Get-Item ($NinjaRegPath | Split-Path -ErrorAction Stop) | Remove-Item -Recurse -Force -ErrorAction Stop
        Write-LogEntry 'Successfully removed.'
    }
    catch {
        Write-LogEntry "Failed to remove key: $($_.Exception.Message)"
        Write-LogEntry 'Continuing with removal...'
    }
}

Write-LogEntry 'Removal script completed. Please review the log if any errors were displayed.'
```

## Followup

Related: [[Ninja - Create List of Devices]]. This entry trails off mid-thought — the command after "or run:" never got written down, so the full sequence isn't captured yet. Worth finishing the note next time this comes up, and maybe turning the disable-prevention + uninstall sequence into a one-liner in Scripts/ once the missing step is confirmed.
