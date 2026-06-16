---
category: how-to
tags:
  - onedrive
  - troubleshooting
date: 2026-05-27
source: "[[27-05-26]]"
publish: true
---
# OneDrive Reset

I don't think this has ever worked for me. Shout to Claude's automated follow up the bottom from the daily processing.

CMD Window

Didn't work:
```
%localappdata%\Microsoft\OneDrive\onedrive.exe
```

```
"C:\Program Files\Microsoft OneDrive\OneDrive.exe" /reset
```

## Followup

These two reset commands didn't resolve the issue — worth tracking down what actually fixed it (or whether it did). A few things worth checking if this comes up again:

- `/reset` kills and restarts OneDrive, but if the issue is auth-related or a corrupt sync database, a full sign-out and back in is usually more effective
- The `%localappdata%` path is the per-user install; the `Program Files` path is the machine-wide install — if both failed, it might not have been a OneDrive binary issue at all
- Check `%localappdata%\Microsoft\OneDrive\logs\` for recent errors to understand what was actually wrong

