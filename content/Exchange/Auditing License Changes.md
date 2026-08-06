---
category: how-to
date: 2026-08-04
publish: true
tags:
  - microsoft365
  - licensing
  - audit
  - powershell
  - exchange
---
# Auditing License Changes for a Silently Dropped Service Plan

A client flagged that a bulk license change looked "off" and asked whether anyone had lost their Exchange archive as a result. This walks through how to reconstruct exactly what a license-change event actually did from Entra audit logs, and how to confirm whether a mailbox archive survived it.

See also: [[M365 Unified Audit Log]], [[Check Archive Mailbox Size]], [[AutoExpandingArchive]]

## The trap: SKU naming doesn't mean what it looks like

Before touching the logs — Microsoft's internal `SkuPartNumber` strings for the Business tier are notoriously misleading:

| SkuPartNumber | Actual product |
|---|---|
| `O365_BUSINESS_PREMIUM` | **Microsoft 365 Business Standard** |
| `SPB` | **Microsoft 365 Business Premium** |
| `O365_BUSINESS_ESSENTIALS` | Microsoft 365 Business Basic |

The string literally containing "PREMIUM" is Standard, and the real Premium SKU has a string ID with no obvious relation to the name at all. Any time you're reading raw audit log SKU names, translate them before drawing conclusions — it's easy to misread a Standard removal as a Premium removal (or vice versa).

## Quick reference — common SKU / plan IDs

Audit logs give you the SKU as a raw GUID as often as they give you the string name, so it's worth having both on hand. These are the ones that come up constantly on SMB tenants:

| Product | String ID | GUID (SkuId) |
|---|---|---|
| Microsoft 365 Business Basic | `O365_BUSINESS_ESSENTIALS` | `3b555118-da6a-4418-894f-7df1e2096870` |
| Microsoft 365 Business Standard | `O365_BUSINESS_PREMIUM` | `f245ecc8-75af-4f8e-b61f-27d8114de5f3` |
| Microsoft 365 Business Premium | `SPB` | `cbdc14ab-d96c-4c30-b9f4-6ada7cdc1d46` |
| Exchange Online (Plan 1) | `EXCHANGESTANDARD` | `4b9405b0-7788-4568-add1-99614e613b69` |
| Exchange Online (Plan 2) — the one with the auto-expanding archive built in | `EXCHANGEENTERPRISE` | `19ec0d23-8335-4cbd-94ac-6050e30712fa` |
| Exchange Online Archiving (add-on, for Basic/Standard/Plan 1 tiers) | `EXCHANGEARCHIVE_ADDON` | `ee02fd1b-340e-4a4b-b355-4a514e4c8943` |
| Exchange Online Kiosk | `EXCHANGEDESKLESS` | `80b2d799-d2ba-4d2a-8842-fb0d0f3a4b82` |
| Microsoft Entra ID P1 | `AAD_PREMIUM` | `078d2b04-f1bd-4111-bbd4-b4b1b354cef4` |
| Office 365 E3 | `ENTERPRISEPACK` | `6fd2c87f-b296-42f0-b197-1e91e994b900` |
| Office 365 E5 | `ENTERPRISEPREMIUM` | `c7df2760-2c81-4ef7-b578-5b5392b571df` |

The Business Standard, Business Premium, and Exchange Online Plan 2 rows above were cross-checked against real tenant audit log data, not just the reference doc. The rest came from Microsoft's [product names and service plan identifiers](https://learn.microsoft.com/en-us/entra/identity/users/licensing-service-plan-reference) page — that page is the source of truth and gets new SKUs added fairly often, so if you're chasing something not listed here, check there first rather than guessing from the product name.

## Step 1 — Pull the audit log

**Entra admin center → Audit logs → filter `Activity = Change user license` → Download → CSV.**

This export has two quirks worth knowing before you rely on it:

1. **Small payloads** land in the normal `TargetNModifiedPropertyN Name/OldValue/NewValue` columns.
2. **Large payloads** (a full SKU's list of ~30-40 service plans) don't fit that structure — Entra instead dumps the entire `targetUpdatedProperties` array as a single JSON string inside an `AdditionalDetailNValue` column (key `b`). You have to detect and unwrap that separately.
3. That JSON string gets **hard-truncated at 10,000 characters per cell**, with no continuation field. For a bulk SKU with a long service-plan list, the tail (the plan-by-plan detail) gets cut off mid-object. Fortunately `AssignedLicense` — the actual SKU-level before/after value you care about — sits near the front of the payload and survives the truncation even when the rest doesn't.

## Step 2 — Parse it properly

Rather than `ConvertFrom-Json` the truncated blob (it'll throw on the cut-off tail), regex out just the `AssignedLicense` old/new arrays directly from the raw text — this survives truncation since it only needs the *start* of the string to be intact.

```powershell
# Parses an Entra audit log CSV export (Activity = "Change user license") into a
# per-user before/after report of which SKUs were added/removed.
#
# Handles two Entra CSV export quirks:
#  - large payloads live in AdditionalDetail "b", not the ModifiedProperty columns
#  - that field truncates at 10,000 chars, so we regex the raw text instead of
#    trying to fully JSON-parse a value that may be cut off mid-object

$csvPath = Read-Host "Path to audit log CSV export"
if (-not (Test-Path $csvPath)) {
    Write-Host "`nFile not found: $csvPath"
    return
}

# SkuPartNumber -> real product name. Extend as needed for your tenant's SKUs.
$skuMap = @{
    'O365_BUSINESS_PREMIUM'    = 'Business Standard (legacy SkuPartNumber - NOT Premium)'
    'SPB'                      = 'Business Premium'
    'O365_BUSINESS_ESSENTIALS' = 'Business Basic'
    'O365_BUSINESS'            = 'Apps for Business'
    'EXCHANGESTANDARD'         = 'Exchange Online Plan 1'
    'EXCHANGEENTERPRISE'       = 'Exchange Online Plan 2'
    'EXCHANGEARCHIVE_ADDON'    = 'Exchange Online Archiving (add-on)'
    'AAD_PREMIUM'              = 'Entra ID P1'
    'STANDARDPACK'             = 'Office 365 E1'
    'ENTERPRISEPACK'           = 'Office 365 E3'
    'ENTERPRISEPREMIUM'        = 'Office 365 E5'
}

function Resolve-SkuName {
    param([string]$SkuName)
    if ($skuMap.ContainsKey($SkuName)) { return $skuMap[$SkuName] }
    return "$SkuName (unmapped - verify in Entra > Licenses)"
}

function Get-SkuNames {
    param([string]$RawValue)
    if ([string]::IsNullOrWhiteSpace($RawValue) -or $RawValue -eq '[]') { return @() }
    $m = [regex]::Matches($RawValue, 'SkuName=([^,\]]+)')
    return $m | ForEach-Object { $_.Groups[1].Value.Trim() }
}

function Get-AssignedLicenseDeltaFromBlob {
    # Unwraps the AdditionalDetail "b" JSON blob and regexes out just the
    # AssignedLicense entry - avoids ConvertFrom-Json failing on a truncated tail.
    param([string]$RawBlob)
    if ([string]::IsNullOrWhiteSpace($RawBlob)) { return $null }
    $clean = $RawBlob -replace '\\"', '"'
    $m = [regex]::Match($clean, '"Name":"AssignedLicense","OldValue":\[(.*?)\],"NewValue":\[(.*?)\]')
    if (-not $m.Success) { return $null }
    [PSCustomObject]@{
        Old = Get-SkuNames $m.Groups[1].Value
        New = Get-SkuNames $m.Groups[2].Value
    }
}

$rows = Import-Csv $csvPath | Where-Object { $_.Activity -eq 'Change user license' }
Write-Host "`nFound $($rows.Count) 'Change user license' events`n"

$report = foreach ($row in $rows) {
    for ($t = 1; $t -le 3; $t++) {
        $upn = $row."Target${t}UserPrincipalName"
        if ([string]::IsNullOrWhiteSpace($upn)) { continue }

        $foundInModifiedProps = $false

        # Standard path: small payloads in the ModifiedProperty columns
        for ($p = 1; $p -le 5; $p++) {
            if ($row."Target${t}ModifiedProperty${p}Name" -ne 'AssignedLicense') { continue }
            $foundInModifiedProps = $true
            $oldSkus = Get-SkuNames $row."Target${t}ModifiedProperty${p}OldValue"
            $newSkus = Get-SkuNames $row."Target${t}ModifiedProperty${p}NewValue"
            $removed = $oldSkus | Where-Object { $newSkus -notcontains $_ }
            $added   = $newSkus | Where-Object { $oldSkus -notcontains $_ }
            if (-not $removed -and -not $added) { continue }
            [PSCustomObject]@{
                DateUTC     = $row.'Date (UTC)'
                User        = $upn
                Removed     = ($removed | ForEach-Object { Resolve-SkuName $_ }) -join '; '
                Added       = ($added   | ForEach-Object { Resolve-SkuName $_ }) -join '; '
                InitiatedBy = $row.ActorUserPrincipalName
                Source      = 'ModifiedProperty'
            }
        }
        if ($foundInModifiedProps) { continue }

        # Fallback path: bulk payloads stuffed into AdditionalDetail "b"
        for ($d = 1; $d -le 6; $d++) {
            if ($row."AdditionalDetail${d}Key" -ne 'b') { continue }
            $delta = Get-AssignedLicenseDeltaFromBlob $row."AdditionalDetail${d}Value"
            if (-not $delta) { continue }
            $removed = $delta.Old | Where-Object { $delta.New -notcontains $_ }
            $added   = $delta.New | Where-Object { $delta.Old -notcontains $_ }
            if (-not $removed -and -not $added) { continue }
            [PSCustomObject]@{
                DateUTC     = $row.'Date (UTC)'
                User        = $upn
                Removed     = ($removed | ForEach-Object { Resolve-SkuName $_ }) -join '; '
                Added       = ($added   | ForEach-Object { Resolve-SkuName $_ }) -join '; '
                InitiatedBy = $row.ActorUserPrincipalName
                Source      = 'AdditionalDetailBlob'
            }
        }
    }
}

$report = $report | Sort-Object User, DateUTC
$outPath = [System.IO.Path]::Combine((Split-Path $csvPath), "LicenseChangeReport_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv")
$report | Export-Csv -Path $outPath -NoTypeInformation
$report | Format-Table DateUTC, User, Removed, Added -AutoSize -Wrap
Write-Host "`nReport saved to: $outPath"
```

## Step 3 — Read the diff, not just the event count

The report groups cleanly into two patterns once you look at Removed vs Added per user:

| Pattern | What it means |
|---|---|
| Same SKU shows in both Removed and Added, seconds apart | A harmless toggle-off/toggle-on — common when an admin clears and reassigns a license to force Exchange/SharePoint to re-provision a stuck service plan. No real change. |
| A SKU shows in Removed but **never reappears** in Added for that user | A genuine loss. This is the one to act on. |

In the incident this note is based on, a bulk relicense touched five mailboxes in the same ~90-second window. Four were clean toggles — same SKU removed and re-added within seconds. One user's event removed **two** SKUs (their base plan *and* a standalone Exchange Online Plan 2 add-on) but only the base plan came back. That standalone add-on is what actually carries the large auto-expanding archive on that tier — losing it silently, with nobody watching the diff, is exactly the kind of thing that goes unnoticed until someone asks "where's my archive?" months later.

## Step 4 — Confirm whether the archive was actually affected

Losing the license doesn't delete the archive mailbox — Microsoft holds it for a grace period (documented around 30 days) before any backend cleanup starts, and a same-session or same-day toggle is far too fast for the backend license-processing job to have acted on it at all (that sync runs on the order of tens of minutes to hours, not seconds). So a quick removal/re-add pair is not a data-loss event, even if you eyeball it and it looks alarming in the audit log.

Check the real state directly rather than trusting the license alone:

```powershell
Get-Mailbox user@clientdomain.com |
  Select-Object DisplayName, ArchiveStatus, ArchiveState, ArchiveGuid, ArchiveQuota, ArchiveWarningQuota
```

- `ArchiveStatus: None` + all-zero `ArchiveGuid` → no archive ever existed on that mailbox. Nothing to worry about, regardless of what the license history shows.
- `ArchiveStatus: Active` + a real `ArchiveGuid` → the archive object still exists and is enabled, independent of current license state. See [[Check Archive Mailbox Size]] for pulling actual archive size/item count once you've confirmed it's present.

In this case the affected user's archive came back showing `ArchiveStatus: Active`, auto-expanding, tens of gigabytes and tens of thousands of items intact — the entitlement had been missing for over a week by the time it was caught, and the archive was untouched. Reassigning the dropped SKU immediately closed the gap before it got anywhere near the actual grace-period boundary.

## Notes

- A quota mismatch (e.g. a primary mailbox showing a bigger quota than its current license entitles) is usually unrelated and harmless — Exchange doesn't automatically shrink a mailbox's quota when a license downgrades, so old, larger quotas can persist indefinitely from a plan the user was on long before your audit log's retention window even starts. Check with `Get-Mailbox | Select UseDatabaseQuotaDefaults, ProhibitSendQuota, ProhibitSendReceiveQuota` if you want to confirm it's a stale override rather than anything to do with the recent change.
- Standard Entra audit log retention (30 days on P1/P2, 7 on Free) means this technique only works for changes inside that window — anything older needs to already be flowing to a Log Analytics workspace. See [[M365 Unified Audit Log]] for retention by license tier.
- After any bulk/group-based license operation, re-run this diff rather than assuming a "successful" change in the admin center means every prior entitlement came back the way it went in.
