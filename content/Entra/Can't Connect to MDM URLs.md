---
category: troubleshooting
tags:
  - entra
  - intune
  - mdm
  - error
  - enrollment
publish: true
---
## Unable to enroll to either Work or School or Entra AD.

Error says it can't connect to the organisation's MDM URLs

Error: invalid_client
Error Subcode:
Description: failed%2to%2authenticate%2user

![[MDMWrong.png]]

The MDM URLs are an Intune thing, if they don't have Intune it will try to connect and fail.

Go to the Entra Admin Panel, search or 'MDM', you'll get a result for "Mobility (MDM and WIP), from there click through into "Microsoft Intune". In the Intune screen, set the MDM user scope to none, this way it will stop trying to enroll into Intune and just do Entra/AzureAD

![[MDMsettings.png]]