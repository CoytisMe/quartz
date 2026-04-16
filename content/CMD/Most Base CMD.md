---
category:
publish: true
tags:
  - cmd
  - windows
  - tips
  - users
---
## Make a User
Two lines to create a new user and give it local admin in a CMD window, only really useful for making admin accounts since Azure took over.
```cmd
net user <username> <password> /add
```

```cmd
net localgroup administrators <username> /add
```

Related: To change a password it’s just:
```cmd		
Net user <username> <password>
```
## Set a computer to not sleep when plugged in:
```cmd
powercfg /x -standby-timeout-ac 0
```
## Force restart a computer
When someone’s getting fired change their password with the above and then hit em with this.
```cmd
shutdown /r /t 0 /f	
```
Note: The /r means restart, the /t 0 means ‘In 0 seconds’ and the /f means force. So if you just want to shut it down you can figure it out

## Scan the file system
```cmd
sfc/scannow
```
It does something I think, mostly it just leaves a processing CMD window on the client screen. Great if you want to get a coffee, nobody questions a processing CMD window.
