---
category: tech
date: 2026-09-03
publish: true
tags:
  - linux
  - gnome
  - ninja-rmm
source: "[[02-09-26]]"
---
## Linux
Fedora/Gnome super key not mapping to Windows key when connected to client's PC via Ninja RMM

Super key is a high level call, needs to be disabled to allow Ninja to grab it and pass it through.
```bash
gsettings set org.gnome.mutter overlay-key ''
```

Slight pain but just use 3 finger swipe up to do same job.

If I want to revert it:
```bash
gsettings set org.gnome.mutter overlay-key 'Super_L'
```

### Notes from the end of the Claude Convo
A couple of notes:

- This is GNOME-specific. If that Thinkpad is actually running KDE Plasma or something else, the fix is different (KDE binds Super to the app launcher/Overview under System Settings → Shortcuts → rather than overlay-key) — let me know if it's not GNOME and I'll give the right steps.
- Fullscreen not working is likely a separate, unrelated grab (often Super isn't involved there — could be the browser/app not getting an F11 equivalent, or Ninja's fullscreen toggle relying on a button rather than a hotkey). Worth checking if Ninja Remote has an on-screen fullscreen button instead of relying on a key combo.
- Glad it worked. If you ever miss it, you can rebind Activities to something else instead of losing it entirely — e.g. Alt+F1 or a corner-hot-spot — rather than going without. Otherwise, all good.

## Followup

On the Alt+F1 question: the Activities/Overview toggle isn't tied to a special key — it's a normal custom shortcut binding, so it can be set to anything. `overlay-key` (the setting disabled above) only controls the tap-Super-to-open behaviour specifically; the actual Overview toggle is a separate keybinding (`org.gnome.shell.keybindings toggle-overview`) that can be rebound in Settings → Keyboard → View and Customize Shortcuts, or directly via:
```bash
gsettings set org.gnome.shell.keybindings toggle-overview "['<Alt>F1']"
```
Any combo works — Alt+F1, a corner hot-spot (separate hot-corner setting), or a spare function key. No restriction to specific keypresses.
