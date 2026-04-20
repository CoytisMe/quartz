---
category: how to
date: 2026-04-20
publish: true
tags:
  - windows
  - edge
  - browser
  - microsoft
---
How to disable the dumbass feature in edge that changes Windows+Shift+S into a screenshot / web search within Edge itself so you can't use the snipping tool.

## Method 1: Regedit

Head to this location:

	Computer\HKEY_CURRENT_USER\Software\Microsoft\Edge

Create a DWORD and name it

	WebCaptureEnabled

Double click that DWORD and set the value to 0
## Method 2: CMD
(It's a different way to do the same thing)

Open CMD, enter this command

```cmd
reg add "HKCU\Software\Microsoft\Edge" /v WebCaptureEnabled /t REG_DWORD /d 0
```

Hit enter.

## Secret Final Step
Realise that you are in fact the dumbass and the key bind for edges image web search thing is Control+Alt+S, it doesn't interfere with Snipping Tool, it was stupid to think it even could, you've wasted your time and you need to learn to use a keyboard.