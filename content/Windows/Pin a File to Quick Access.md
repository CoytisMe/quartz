---
category: reference
date: 2026-04-20
publish: true
tags:
  - powershell
  - quick-access
  - windows
  - tips
---
## Learned this because LEAP sucks

Pin a folder to someone's quick access via terminal, I was using it pin people's LEAP PDF cache while Adobe was broken.

```powershell
$shell = New-Object -ComObject Shell.Application
$folder = $shell.Namespace("$env:LOCALAPPDATA\path\to\file")
$folder.Self.InvokeVerb("pintohome")
```

If you don't want local app data just replace it...