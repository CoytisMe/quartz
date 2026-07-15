---
category: work-note
tags:
  - windows
  - keyboard
  - registry
  - drivers
  - bios
  - lockout
date: 2026-07-15
publish: true
---
# Keyboard Code 19 - Orphaned Filter Driver

Symptom: no keyboards work anywhere (multiple keyboards, all USB ports), mouse fine, gets to Windows login screen. Found on an HP Z1 handed back by outgoing client IT with no credentials given — worth checking for on any lockout/unknown-history handover, not just this fault.

## Check 1 — BIOS port hiding

Some BIOS builds have a **Port Settings** section (Security tab) that can hide individual USB ports from the OS entirely. Tooltip usually explains it. If checked, the OS never sees the device at all — explains keyboard-dead-but-mouse-works if only some ports were hidden.

Fix: uncheck the hidden ports. If mouse also drops out after changing this, don't panic — it's recoverable via a CMOS/BIOS reset (power drain 20-30s, or clear CMOS jumper/button) rather than a hardware fault.

## Check 2 — Device Manager Code 19

`Windows cannot start this device because its configuration information (in the registry) is incomplete or damaged` (Code 19) on all keyboards = corrupted/orphaned filter driver in the registry, not a hardware fault.

Look at:
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4D36E96B-E325-11CE-BFC1-08002BE10318}
```
(Keyboard class GUID). Check the `UpperFilters` and `LowerFilters` values (REG_MULTI_SZ). Default/healthy value is just `kbdclass`. Anything else listed above/below it is a filter driver in the load chain — if that driver's binary is missing, the whole chain fails and **no keyboard works, at all, regardless of port or device**.

Same check applies to Mouse class if that's affected too:
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4D36E96F-E325-11CE-BFC1-08002BE10318}
```

On the HP Z1 case, `UpperFilters` was `HPKbfDriver kbdclass` — HP's keystroke-encryption filter (HP Client Security Manager / Sure Sense, anti-keylogger feature on HP business machines). No `HPKbfDriver.sys` existed anywhere on disk — the security software had been uninstalled (or the machine decommissioned) without cleanly removing the filter reference, orphaning it.

Fix: export the registry key first (evidence trail + rollback), then edit `UpperFilters` to remove the dead driver name, leaving just `kbdclass`. Reboot. Also worth checking Programs and Features for leftover HP Client Security / Sure Sense / Wolf Security entries.

If keyboard doesn't work well enough to navigate regedit, use the on-screen keyboard (Ease of Access icon, bottom-right of login screen) with the mouse, or edit the hive offline via a WinPE tool (Hiren's/Gandalf — File > Load Hive on the offline `SYSTEM` hive).

## Bigger picture

Not necessarily malicious — this pattern (BIOS port hiding + orphaned security-driver filter) reads more like a botched decommission/uninstall than deliberate sabotage. Still worth documenting (screenshots, dates) as part of the handover record when outgoing IT hasn't provided credentials.
