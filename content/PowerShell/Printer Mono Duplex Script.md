---
category: tech
date: 2026-09-04
publish: true
tags:
  - powershell
  - printing
source: "[[04-09-26]]"
---
## Set Printer Settings with Script

```powershell
$PrinterName = "Your-Ricoh-Printer-Name"
```
```powershell
Set-PrintConfiguration -PrinterName $PrinterName -Color $false -DuplexingMode "OneSided"
```

### Scott's Script for Indeed's Printers
Didn't work
```powershell
<#
.SYNOPSIS
    Forces the default print configuration for all installed printers to Monochrome (B&W).

.DESCRIPTION
    Iterates every printer on the machine and sets its default PrintConfiguration
    Color property to $false using the built-in PrintManagement module.
    Skips virtual/software "printers" that don't have real color settings
    (PDF writers, XPS, Fax, OneNote, etc.) since setting color on those either
    fails or does nothing useful.

    NOTE: This sets the *default* the driver hands to applications. It is not an
    absolute hard block — an application (or user) that explicitly requests color
    in the print dialog can still override it, and this depends on the driver
    correctly honoring the DEVMODE color flag. In practice the vast majority of
    business printer drivers (HP, Brother, Kyocera, Xerox, Ricoh, Canon, etc.)
    respect this setting reliably.

.NOTES
    Run as SYSTEM / elevated (e.g. via NinjaOne script deployment) so it applies
    to machine-wide printer objects rather than just the interactively logged
    on user's mapped printers.
#>

[CmdletBinding()]
param(
    # Add any printer name patterns here you want to skip (wildcards allowed)
    [string[]]$ExcludePatterns = @(
        'Microsoft Print to PDF',
        'Microsoft XPS Document Writer',
        'Fax',
        'OneNote*',
        'Send To OneNote*',
        'Adobe PDF'
    )
)

$results = [System.Collections.Generic.List[pscustomobject]]::new()

# Make sure the PrintManagement module is available (built into Windows 10/11 & Server)
if (-not (Get-Module -ListAvailable -Name PrintManagement)) {
    Write-Error "PrintManagement module not found on this system. Aborting."
    exit 1
}
Import-Module PrintManagement -ErrorAction Stop

$printers = Get-Printer | Where-Object {
    $name = $_.Name
    -not ($ExcludePatterns | Where-Object { $name -like $_ })
}

if (-not $printers) {
    Write-Output "No eligible physical printers found."
    exit 0
}

foreach ($printer in $printers) {
    $entry = [pscustomobject]@{
        PrinterName = $printer.Name
        Status      = ''
        Detail      = ''
    }

    try {
        # Check current config first (some drivers don't expose Color at all)
        $config = Get-PrintConfiguration -PrinterName $printer.Name -ErrorAction Stop

        if ($null -eq $config.Color) {
            $entry.Status = 'Skipped'
            $entry.Detail = 'Driver does not expose a Color setting'
        }
        elseif ($config.Color -eq $false) {
            $entry.Status = 'AlreadyMono'
            $entry.Detail = 'Already set to monochrome'
        }
        else {
            Set-PrintConfiguration -PrinterName $printer.Name -Color $false -ErrorAction Stop
            $entry.Status = 'Updated'
            $entry.Detail = 'Set to monochrome'
        }
    }
    catch {
        $entry.Status = 'Failed'
        $entry.Detail = $_.Exception.Message
    }

    $results.Add($entry)
}

$results | Format-Table -AutoSize | Out-String | Write-Output

$failed = $results | Where-Object { $_.Status -eq 'Failed' }
if ($failed) {
    Write-Warning "$($failed.Count) printer(s) failed to update. See table above."
    exit 1
}

Write-Output "Done. $($results.Count) printer(s) evaluated."
```

## Followup

Scott's script hits every printer via `Get-Printer`/`PrintManagement`, which only touches locally-installed printer *objects* — if Indeed's printers are shared/network queues that this machine doesn't have locally installed (or if the driver reports `Color` as `$null`, hitting the `Skipped` branch), the script would run clean but silently do nothing. Worth re-running with the try/catch removed temporarily, or just checking `Get-PrintConfiguration` output per-printer first, to see whether it's actually landing in `Skipped` vs `Failed` vs not finding the printers at all.

The one-liner version above sets `DuplexingMode` too, which Scott's loop doesn't touch — if the goal for Indeed is mono *and* duplex, that's a second gap even if the color part gets sorted.
