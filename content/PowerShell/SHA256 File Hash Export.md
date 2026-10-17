---
category: tech
date: 2026-10-05
publish: true
tags:
  - powershell
  - purview
  - audit
source:
---
## Learning this made me feel important

Hash exported files (e.g. Purview audit log CSV exports) so you can show later that they haven't changed since export.

## Quick one-liner
Hashes every CSV in the current folder:
```powershell
Get-FileHash .\*.csv -Algorithm SHA256 | Export-Csv hashes.csv -NoTypeInformation
```

## Script with timestamps
Put files in `C:\Users\Rick\hashes\files`. Hashes go to a CSV in `C:\Users\Rick\hashes`.
```powershell
# Set the folder path and output location
$FolderPath = "C:\Users\Rick\hashes\files"
$Timestamp  = Get-Date -Format "yyyyMMdd_HHmmss"
$CSVPath    = "C:\Users\Rick\hashes\HashOutput.csv"

# Get files, calculate SHA256 hashes, and add date/time
Get-ChildItem -Path $FolderPath -Recurse -File | ForEach-Object {
    $HashResult = Get-FileHash -Path $_.FullName -Algorithm SHA256
    [PSCustomObject]@{
        Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        Path      = $_.FullName
        Algorithm = $HashResult.Algorithm
        Hash      = $HashResult.Hash
    }
} | Export-Csv -Path $CSVPath -NoTypeInformation

Write-Host "Hashes exported successfully to $CSVPath"
```

Related: [[M365 Unified Audit Log]], [[PowerShell Tips and Scripts]]

## Followup
- `$Timestamp` is set but never used. Probably meant to go in the output name (`HashOutput_$Timestamp.csv`) so each run doesn't overwrite the last one. Worth fixing if this is evidence-of-custody work.
- If Drive/OneDrive sync gives an "in use" error on the file, hash from a local, non-synced copy.
