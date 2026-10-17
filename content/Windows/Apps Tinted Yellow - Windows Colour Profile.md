---
category: tech
date: 2026-10-17
publish: true
tags:
  - kbdraft
  - publickb
  - windows
  - display
  - troubleshooting
source:
---
# Apps Tinted Yellow - Windows Colour Profile

## Symptom
Chrome, Edge and Teams are tinted yellow, while the rest of Windows looks normal.

## Cause
Colour management. The Windows setting that lets Windows manage colour for apps had been switched off.

## Fix
Settings > System > Display > (the display) > **Colour profile**. Turn back on the option that lets Windows automatically manage colour for apps.

## What didn't work
- Forcing the colour profile through chrome://flags and edge://flags. Teams stayed yellow, so this isn't the real fix.
