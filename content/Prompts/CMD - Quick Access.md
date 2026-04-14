---
category: how-to
tags:
  - cmd
  - windows
  - file-explorer
  - quick-access
  - registry
---
### Option 1

Your Quick Access folder is probably corrupt, the best option is to delete the content of Quick Access, and then re-pin the folders you need there.

Click your Start Button, then just type cmd and on the resulting menu, right click Command Prompt and select 'Run as Administrator'

Paste this command into Command Prompt and press Enter:

- del /F /Q %APPDATA%\Microsoft\Windows\Recent\*

Paste this command into Command Prompt and press Enter:

- del /F /Q %APPDATA%\Microsoft\Windows\Recent\AutomaticDestinations\*

Paste this command into Command Prompt and press Enter:

- del /F /Q %APPDATA%\Microsoft\Windows\Recent\CustomDestinations\*

Then, close Command Prompt and restart (not shut down) your PC.


### Option 2

Quick Access may be corrupt, try this fix, this will remove all shortcuts from the Quick Access area in File Explorer, but not the actual files.

Open File Explorer.

In the Address bar paste this and press Enter:

shell:recent\AutomaticDestinations

Delete the contents of the resulting folder

In the Address bar paste this and press Enter:

shell:recent\CustomDestinations

Delete the contents of the resulting folder

Restart (not shut down) your PC.


Computer\HKEY_CURRENT_USER\Software\Classes\CLSID\{48782065-C5A6-4245-BAAB-8CDBF8DCC7E9}


## Reg Keys

**Quick Access**
Computer\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders

**NameSpace (Dropbox etc)**
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Desktop\NameSpace