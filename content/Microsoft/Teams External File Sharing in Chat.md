---
category: tech
date: 2026-10-13
publish: true
tags:
  - m365
  - teams
  - powershell
source:
---
# Teams External File Sharing in Chat

Allow file sharing in chat with external users.

Need PowerShell Teams Module

```powershell
Set-CsTeamsFilesPolicy -Identity Global -FileSharingInChatswithExternalUsers Enabled
```

Check with:

```powershell
Get-CsTeamsFilesPolicy -Identity Global
```

Related: [[External Access to SharePoint - No MS Account]], [[PowerShell Tips and Scripts]]

## Followup
- The module is `MicrosoftTeams` (`Install-Module MicrosoftTeams`, then `Connect-MicrosoftTeams` with a Teams admin account) before the `CsTeamsFilesPolicy` cmdlets will load.
- If a tenant runs Huntress ISPM, check it isn't going to revert this.
