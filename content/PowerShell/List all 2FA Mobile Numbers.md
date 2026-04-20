---
category: script
tags:
  - powershell
  - entra
  - mfa
  - users
  - graph-api
publish: true
---
This will give you a CSV of what mobile number each account is using for 2FA
```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "User.Read.All","UserAuthenticationMethod.Read.All"

# Get all users
$users = Get-MgUser -All

# Iterate through users and get phone methods
$results = foreach ($user in $users) {
    $phoneMethods = Get-MgUserAuthenticationPhoneMethod -UserId $user.Id
    foreach ($method in $phoneMethods) {
        [PSCustomObject]@{
            UserPrincipalName = $user.UserPrincipalName
            DisplayName       = $user.DisplayName
            PhoneType         = $method.PhoneType
            PhoneNumber       = $method.PhoneNumber
        }
    }
}

# View results or export to CSV
$results | Format-Table -AutoSize
$results | Export-Csv -Path "C:\Temp\MFA_Phone_Numbers.csv" -NoTypeInformation
```
