---
category: how-to
date: 2026-06-04
publish: true
tags:
  - microsoft365
  - audit
  - compliance
  - powershell
---

# M365 Unified Audit Log (UAL)

See also: [[M365 Entra Diagnostic Logging]]

## Quick Commands

**Connect:**
```powershell
Connect-ExchangeOnline
```

**Check if UAL is enabled:**
```powershell
Get-AdminAuditLogConfig | Select-Object UnifiedAuditLogIngestionEnabled
# or
Get-OrganizationConfig | Select-Object AuditDisabled
# AuditDisabled: False = ON | True = OFF
```

**Enable UAL:**
```powershell
Set-AdminAuditLogConfig -UnifiedAuditLogIngestionEnabled $true
```
Takes up to 60 minutes to kick in. Not retroactive.

**Module required:** `ExchangeOnlineManagement`
```powershell
Install-Module ExchangeOnlineManagement -Scope CurrentUser
```

---

## General Activity Report — Avg Log Entries Per User Per Day

Returns average UAL entries per user per day over a given period. Good for spotting inactive accounts or unusual activity levels.

```powershell
# Parameters
$StartDate = (Get-Date).AddDays(-30)   # Adjust lookback period here
$EndDate   = Get-Date
$Days      = ($EndDate - $StartDate).Days

# Pull UAL for the period (max 5000 results per call — loop if needed for busy tenants)
$Results = Search-UnifiedAuditLog -StartDate $StartDate -EndDate $EndDate -ResultSize 5000

# Group by user and calculate daily average
$Results |
    Group-Object UserIds |
    Select-Object `
        @{N='User';       E={ $_.Name }},
        @{N='TotalEvents';E={ $_.Count }},
        @{N='AvgPerDay';  E={ [math]::Round($_.Count / $Days, 1) }} |
    Sort-Object TotalEvents -Descending |
    Format-Table -AutoSize
```

> **Note:** `Search-UnifiedAuditLog` returns max 5000 results per call. For busy tenants or long date ranges, loop with `-SessionCommand ReturnNextPresetNumberOfResults` to get all records.

**Loop version for large result sets:**
```powershell
$StartDate  = (Get-Date).AddDays(-30)
$EndDate    = Get-Date
$Days       = ($EndDate - $StartDate).Days
$AllResults = @()
$Session    = [System.Guid]::NewGuid().ToString()

do {
    $Batch = Search-UnifiedAuditLog -StartDate $StartDate -EndDate $EndDate `
                -SessionId $Session -SessionCommand ReturnNextPresetNumberOfResults `
                -ResultSize 5000
    $AllResults += $Batch
} while ($Batch.Count -gt 0)

$AllResults |
    Group-Object UserIds |
    Select-Object `
        @{N='User';       E={ $_.Name }},
        @{N='TotalEvents';E={ $_.Count }},
        @{N='AvgPerDay';  E={ [math]::Round($_.Count / $Days, 1) }} |
    Sort-Object TotalEvents -Descending |
    Format-Table -AutoSize
```

---

## Overview

The Unified Audit Log is the main activity log for M365. Covers:
- **Exchange** — mail send/receive/delete, mailbox access, delegates
- **SharePoint & OneDrive** — file access, sharing, downloads
- **Teams** — messages, meetings, channel activity
- **Entra** — sign-ins, role changes, app consent, MFA events
- **Other** — Power BI, Forms, Planner, etc.

Search via **Microsoft Purview → Audit** or `Search-UnifiedAuditLog` in PowerShell.

---

## Retention by Licence

| Licence                   | UAL Retention |
| ------------------------- | ------------- |
| Business Basic / Standard | 90 days       |
| **Business Premium**      | **180 days**  |
| E3                        | 90 days       |
| E5 / Purview add-on       | 1 year+       |

For longer retention, export to **Log Analytics** (costs money on ingestion but enables long-term KQL queries and Sentinel integration).

---

## Notes

- Older tenants (pre-2019) may have UAL off by default — always check on new client onboarding
- Enabling is not retroactive — no history before it's turned on
- Mailbox audit logging (who accessed whose mailbox) is separate and on by default since 2019
- For incident response without UAL you have basically nothing — always enable it
- Both Coytis and Greenhood tenants confirmed **ON** as of 2026-06-04
