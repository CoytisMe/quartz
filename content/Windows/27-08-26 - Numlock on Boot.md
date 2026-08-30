---
category: reference
tags:
  - windows
  - registry
  - keyboard
date: 2026-08-27
publish: false
source: "[[Dailys/2026/08/27-08-26]]"
---
# Set Numlock State on Boot

**Regedit**
`HKEY_USERS\.DEFAULT\Control Panel\Keyboard`

Set `InitialKeyboardIndicators` to `2` (or `2147483650` depending on the OS build) to force Numlock on at boot/login screen.

## Followup

`HKEY_USERS\.DEFAULT` is the default profile template — this sets Numlock on for the login screen and any *new* user profile created afterward. Existing user profiles have their own `HKEY_CURRENT_USER\Control Panel\Keyboard` copy and won't pick this up retroactively; same key needs setting under each existing user's hive (or `HKCU` directly when logged in as them) if the goal is "every account, everywhere."
