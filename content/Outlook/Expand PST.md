---
category: how-to
tags:
  - outlook
  - pst
  - registry
  - storage
publish: true
---
1. Open Registry Editor
	Start Menu > Regedit or Windows+R and then regedit
	

2. Either navigate to the below key with the folders on the side of copy paste the full path into the top
   
		Computer\HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Outlook\PST
   
4. Right click on the white space and hit New > DWORD

5. Give it a name (Below), hit enter.
   
6. Double click it and enter the required value (again, below)
		Make sure it's set to decimal
		
7. Done

## Values in question:

WarnLargeFileSize = 95000
	The size, in MB (95gb) that the PST will get to before it tells you it's almost full.

MaxLargeFileSize = 100000
	The size, in MB (100gb) at which the PST will stop working.





```cmd
reg add "HKCU\Software\Microsoft\Office\16.0\Outlook\PST" /v WarnLargeFileSize /t REG_DWORD /d 95000 /f

reg add "HKCU\Software\Microsoft\Office\16.0\Outlook\PST" /v MaxLargeFileSize /t REG_DWORD /d 100000 /f
```

Script Saved in Ninja as automation (Run as Local User)

