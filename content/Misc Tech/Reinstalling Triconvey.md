---
category: how-to
date: 2026-06-16
publish: true
tags:
  - triconvey
  - conveyancing
  - 3rdparty
  - uninstall
---
In the actual event Triconvey breaks, ie the 'logging out of account that don't exist' error.

You gotta reinstall with both hands.

1. Ensure all Microsoft office products have been closed (Word/Outlook/Excel etc).

2. Perform a DB file backup and save it to a stable location.

3. Open Task Manager and suspend the Smokeball Update Service and Windows Service.

4. Open the Windows Task Tray and close the Smokeball Tray Icon from there. Otherwise, exit the Smokeball process from the Task Manager.

5. Delete C:\Program Files\Smokeball\
   
6. Delete %localappdata%\Smokeball

7. Uninstall triConvey from the Windows Apps & features window.


## To reinstall triConvey:

[Download triConvey.](https://www.trisearch.com.au/triconvey-download/)

Reinstall triConvey and log the user back into their account once the installation is complete. Ensure they can access their account.