---
category: how-to
tags:
  - powershell
  - exchange
  - compliance
  - holds
  - recoverable-items
publish: true
---
Symptoms:
- "Unable to Delete Items"
- Deleted items are being restored automatically

Step 1:
**Check size of the recoverable items folder**
- get-mailboxfolderstatistics -identity *user* -folderscope recoverableitems | FL name,folderpath,folderandsubfoldersize,storagequota


Step 2:
**Check for Holds**
- MS guide for identifying holds
	- https://learn.microsoft.com/en-us/purview/ediscovery-identify-a-hold-on-an-exchange-online-mailbox

Step 2b:
**No holds found but DiscoveryHolds folder is still huge?**
- Seen this: `get-mailbox *user* | FL *hold*` comes back all False/empty (LitigationHoldEnabled, RetentionHoldEnabled, InPlaceHolds all clear), yet DiscoveryHolds is carrying most of the Recoverable Items size.
- This means a hold *was* on the mailbox at some point (litigation hold, eDiscovery case hold, or a retention policy) and got removed, but the content already pushed into DiscoveryHolds never got purged — normal Managed Folder Assistant cycles don't clean that folder on their own.
- Confirm MRM isn't blocked: `get-mailbox *user* | FL RetentionPolicy,ElcProcessingDisabled` — want a policy assigned and ElcProcessingDisabled False.
- Fix is a targeted MFA run with the hold cleanup switch:
```powershell
Start-ManagedFolderAssistant -Identity *user* -HoldCleanup
```
- This forces MRM to re-evaluate everything in DiscoveryHolds against *current* holds and purge anything no longer covered. It's throttled/async — large folders (50GB+) can take a day or more to fully process. Re-check with the Step 1 stats command after.
- Before purging, worth a quick sanity check in Purview audit logs to confirm what hold was previously applied and that it's genuinely gone org-side, not just cleared from this mailbox's view.

Step 2c:
**Ran -HoldCleanup but size isn't moving at all (not even slowly)?**
- Check whether MRM/ELC is actually managing to run against the mailbox:
```powershell
Export-MailboxDiagnosticLogs -Identity *user* -ComponentName MRM
```
- Seen this come back with a `ResourceUnhealthyException: DiskLatency(...)` error, e.g.:
```
Microsoft.Exchange.WorkloadManagement.ResourceUnhealthyException: Resource
'DiskLatency(Guid:... Name:AUSP282DG176-db129 Volume:<unknown>)' is unhealthy and shouldn't be accessed.
   at ...HoldCleanupEnforcer.CollectItemsInFolder(...)
```
- This means ELC's own health monitor is refusing to let it touch the mailbox store because the **backend database is reporting disk latency issues on Microsoft's side**. Every cleanup attempt aborts before it collects anything - not throttling, a hard backend block. Re-running `-HoldCleanup` will keep failing the same way.
- This is not fixable from Exchange Online PowerShell. Raise a Microsoft support ticket, quoting the exact exception text, database name/GUID, and that it's blocking ELC's HoldCleanup pass on the Recoverable Items folder. Gives support something concrete to act on (checking/rebalancing that specific database) instead of a vague "recoverable items too big" ticket.
- Worth checking if the log shows this recurring across multiple db names over time (mailbox moved between databases) - suggests an intermittent backend health issue rather than a single bad disk, useful context for the ticket.

Step 2d:
**`Get-Mailbox | FL *hold*` comes back completely clean, but a hold is still active?**
- `InPlaceHolds` on the mailbox object mostly reflects explicit/legacy assignment. A retention/hold policy scoped to `ExchangeLocation: {All}` (org-wide) can be active and holding everything without ever showing up there.
- Check at the org level instead:
```powershell
Get-OrganizationConfig | FL InPlaceHolds
```
Then match the GUID it returns against:
```powershell
Get-RetentionCompliancePolicy *GUID* -DistributionDetail | FL Name,*Location
```
- If it's genuinely applying org-wide, get the retention rule behind it - this tells you if it can ever purge anything:
```powershell
Get-RetentionComplianceRule -Policy "*PolicyName*" | FL Name,RetentionDuration,RetentionComplianceAction,ExpirationDateOption
```
`RetentionComplianceAction: Keep` + `RetentionDuration: Unlimited` = nothing under this policy is ever eligible to purge, full stop - fixing a backend fault (see Step 2c) won't shrink anything if this is the actual blocker.
- **Figure out if the policy is legitimate or leftover cruft** before touching it:
```powershell
Get-RetentionCompliancePolicy "*PolicyName*" | FL *
```
Check `WhenCreated`/`CreatedBy` vs `WhenChanged`/`LastModifiedBy` - they can be years apart (a recent edit doesn't mean recent creation). Try to resolve `CreatedBy` against a real identity:
```powershell
Get-User -Identity *GUID*
Get-ServicePrincipal *GUID*
```
- If the GUID doesn't resolve to any current user or service principal, and the policy is old (years, not months), that's a strong signal it's leftover migration/setup cruft rather than a deliberate active compliance requirement - the identity that created it is long gone (departed staff, a previous IT provider, or migration tooling), so there's no one left to ask.
- If you're confident nothing currently needs org-wide indefinite retention: disable first (reversible), don't jump straight to delete:
```powershell
Set-RetentionCompliancePolicy -Identity "*PolicyName*" -Enabled $false
```
Watch a day, then remove if nothing breaks:
```powershell
Remove-RetentionCompliancePolicy -Identity "*PolicyName*"
```
Then re-run HoldCleanup so MRM re-evaluates against the now-reduced hold set.

Step3:
**Clear it out.**
I found two methods, one seems a lot more aggressive and complicated than the other.

- a) The Simple slow one, this was tested against a global compliance policy identified in step 2, the below guide is simply exempting from the policy, setting your own retention and letting it process. Steps like removing delay holds and single item retention may still be needed and are in the guide for b.
	- https://techcommunity.microsoft.com/t5/exchange/how-to-clear-the-discovery-holds-folder/m-p/3694295
- b) The more aggressive one, remove all hold, kill user access and use a compliance search script to directly purge it. I haven't done this one. Great info on holds and how to remove then though.
	- https://learn.microsoft.com/en-us/purview/ediscovery-delete-items-in-the-recoverable-items-folder-of-mailboxes-on-hold

Step 4: 
**Wait and Check**
- Check with step one command



**Useful commands**

Get recoverable items data
```powershell
get-mailboxfolderstatistics -identity *user* -folderscope recoverableitems | FL name,folderpath,folderandsubfoldersize,storagequota
```

Search for basically all holds
```powershell
get-mailbox *user* | FL *hold*
```

See the global compliance policies
```powershell
Get-RetentionCompliancePolicy
```

Look up the GUID from the inplace results to find the name and details of a global policy
```powershell
Get-RetentionCompliancePolicy *GUID with no prefix or suffix* -DistributionDetail | FL Name,*Location
```

Exempt a user from the policy
```powershell
Set-RetentionCompliancePolicy -Identity *GUID with no prefix or suffix*  -AddExchangeLocationException *user*
```

Find and remove Delay Holds (There are applied when you change a hold ie exempt from above policy)
```powershell
Get-Mailbox *user* | FL DelayHoldApplied,DelayReleaseHoldApplied
```
```powershell
Set-Mailbox *user* -removedelayholdapplied
```

Turn off single item recovery (need to purge) and change retention period after excluding 
```powershell
set-mailbox *user* -singleitemrecoveryenabled $false
```
```powershell
set-mailbox -identity *user* -retaindeleteditemsfor 10
```
```powershell
get-mailbox *user* | Format-List SingleItemRecoveryEnabled,RetainDeletedItemsFor	
```

Start Processing once new policies are applied
```powershell
Start-Managedfolderassistant -identity *user*
```

Force cleanup of orphaned DiscoveryHolds content (no active hold, but folder still full - see Step 2b)
```powershell
Start-ManagedFolderAssistant -Identity *user* -HoldCleanup
```

A7D92094494754488AEB6819B0604F6D00000A2EB92C0000