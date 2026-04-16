---
category: script
tags:
  - powershell
  - entra
  - licensing
  - users
  - graph-api
publish: true
---
For list only showing licensed or not

```
Connect-Graph -Scopes User.Read.All, Organization.Read.All
```
  
```
Get-MgUser -Filter 'assignedLicenses/$count ne 0' -ConsistencyLevel eventual -CountVariable licensedUserCount -All -Select UserPrincipalName,DisplayName,AssignedLicenses | Format-Table -Property DisplayName,UserPrincipalName
```
  
Export CSV is fucked, but copy paste into Excel works if you change the format-table property to be one at a time

