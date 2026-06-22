---
category: work-note
tags:
  - exchange
  - purview
  - retention
  - powershell
  - mrm
date: 2026-04-22
publish: true
---
# MRM Retention Tags & Policies — PowerShell Reference

## View existing setup

```powershell
# List all retention policies
Get-RetentionPolicy

# List all retention tags
Get-RetentionPolicyTag

# List only archive tags
Get-RetentionPolicyTag | Where-Object {$_.RetentionAction -eq "MoveToArchive"}

# See all tags linked to a specific policy
Get-RetentionPolicy "Default MRM Policy" | Select -ExpandProperty RetentionPolicyTagLinks
```

---

## Create a retention tag

```powershell
# 1 year move to archive
New-RetentionPolicyTag "1 Year Archive" -Type All -RetentionEnabled $true -AgeLimitForRetention 365 -RetentionAction MoveToArchive

# 2 year move to archive
New-RetentionPolicyTag "2 Year Archive" -Type All -RetentionEnabled $true -AgeLimitForRetention 730 -RetentionAction MoveToArchive
```

---

## Create a policy from scratch

```powershell
New-RetentionPolicy "1 Year Archive Policy" -RetentionPolicyTagLinks "1 Year Archive"
New-RetentionPolicy "2 Year Archive Policy" -RetentionPolicyTagLinks "2 Year Archive"
```

---

## Clone Default MRM Policy with swapped archive tag

Use this when you want the same tags as Default MRM Policy but with 1 Year Archive instead of 2 Year.

```powershell
# First — confirm the exact name of the 2 year archive tag in the policy
Get-RetentionPolicy "Default MRM Policy" | Select -ExpandProperty RetentionPolicyTagLinks

# Create the 1 year archive tag if it doesn't exist
New-RetentionPolicyTag "1 Year Archive" -Type All -RetentionEnabled $true -AgeLimitForRetention 365 -RetentionAction MoveToArchive

# Grab all existing tags except the 2 year archive one
$existingTags = (Get-RetentionPolicy "Default MRM Policy").RetentionPolicyTagLinks
$newTags = $existingTags | Where-Object {$_ -notlike "*2 Year*"}

# Create new policy with swapped tag
New-RetentionPolicy "1 Year Archive Policy" -RetentionPolicyTagLinks ($newTags + "1 Year Archive")
```

---

## Add a tag to an existing policy

```powershell
$tag = (Get-RetentionPolicyTag "1 Year Archive").Guid
Set-RetentionPolicy "Default MRM Policy" -RetentionPolicyTagLinks @{Add=$tag}
```

---

## Assign a policy to a mailbox

```powershell
Set-Mailbox -Identity "user@domain.com" -RetentionPolicy "1 Year Archive Policy"
```

---

## Notes

- Run `Enable-OrganizationCustomization` if cmdlets fail — though if it says already enabled, you're fine
- Purview UI may throw a `DirectInvokeCmdletExecutionException` on first-time org setup — PowerShell bypasses this
- Default MRM Policy applies to all mailboxes unless overridden at the mailbox level
