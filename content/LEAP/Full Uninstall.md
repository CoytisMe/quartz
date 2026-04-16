---
category: how-to
tags:
  - leap
  - uninstall
  - reinstall
  - troubleshooting
---
## 1. Uninstall and Kill Processes
Uninstall LEAP via appwiz.cpl

For some reason it will commonly leave stuff behind and leave services running. Go to task manager, search LEAP in ProCesses and Details, kill everything.

Then go to services, search LEAP and stop everything
   
## 2. Remove old files
Use Windows+ R to go to the following locations and delete or rename the following files if you find them.

**%appdata%**
- Delete if found
	- 4D
	- LEAP Desktop
	- LEAP Legal Software

**%localappdata%**
- Delete if found
	- 4D
	- LEAP Office Installations
	- LEAP Legal
  
- Rename 'LEAP Desktop' to 'LEAP Desktop.old'
  
**%localappdata%/Microsoft Corporation**
- Delete any LEAP Folder

**%programdata%**
- Rename 'LEAP Office' to 'LEAP Office.old'

**%programfiles**
- Delete if found
	- LEAP Office
	- LEAP Legal Software

**%temp%**
- Delete if found
	- 4D
	- LEAP
	- LEAP Legal
	- LEAP Cloud
	- LEAP Desktop
## 3. Reinstall LEAP
- Download latest version of LEAP Desktop / LEAP Conveyancer
- **DO NOT LAUNCH IT**
- Repair Office 
	- appwiz.cpl > Office > Repair
- Restart PC
- Launch LEAP
