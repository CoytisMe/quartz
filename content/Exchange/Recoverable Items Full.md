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

A7D92094494754488AEB6819B0604F6D00000A2EB92C0000