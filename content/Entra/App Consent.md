---
category: how-to
tags:
  - entra
  - azure
  - consent
  - permissions
publish: true
---
### NOTE!
Check the consent requests even if you signed the consent on the user's computer with admin creds, there were still some in there.
### Direct Link to Consent

If the consent permissions are not sticking / generally being a pain, use the this link.

> https://login.microsoftonline.com/{tenant-id}/adminconsent?client_id={app-client-id}

{Tenant-ID} = Entra Home
{App-ID} = Specific App Home

![[App Consent Draw.png]]
