---
category: how-to
tags:
  - outlook
  - registry
  - new-outlook
---
However, if the "New Outlook" button disappeared, try running Regedit.exe and go here (or the same with 15.0 if you don't have 16.0):

Computer\HKEY_CURRENT_USER\SOFTWARE\Microsoft\Office\16.0\Outlook\Preferences

Create DWORD Called UseNewOutlook

Set to 0 (decimal)


Computer\HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Outlook\Options\General

REG_DWORD "HideNewOutlookToggle"

Set to 1 (decimal)