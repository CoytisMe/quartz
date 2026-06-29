---
category: tech
date: 2026-06-29
publish: true
tags:
  - powershell
  - entra
  - licensing
  - users
  - graph-api
---
For list only showing licensed or not

```powershell
Connect-Graph -Scopes User.Read.All, Organization.Read.All
```
  
```powershell
Get-MgUser -Filter 'assignedLicenses/$count ne 0' -ConsistencyLevel eventual -CountVariable licensedUserCount -All -Select UserPrincipalName,DisplayName,AssignedLicenses | Format-Table -Property DisplayName,UserPrincipalName
```
  
Export CSV is fucked, but copy paste into Excel works if you change the format-table property to be one at a time

There a version of this that works in [[Script Commands]]

also... [[List all 2FA Mobile Numbers]]
