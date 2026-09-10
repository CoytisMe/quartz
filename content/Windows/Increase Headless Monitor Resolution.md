---
category: reference
date: 2026-09-07
publish: true
tags:
  - windows
  - display
source: "[[Dailys/2026/09/07-09-26]]"
---
**Regedit**
`Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\GraphicsDrivers\Configuration`

Look for a key containing `NOEDID`, `SIM`, or `MSBDD`.

Change:
- `PrimSurfSize.cx` for X
- `PrimSurfSize.cy` for Y

### Virtual Display Driver on GitHub for when the above doesn't work
https://github.com/VirtualDrivers/Virtual-Display-Driver
