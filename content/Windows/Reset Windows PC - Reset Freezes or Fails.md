---
category: how-to
date: 2026-09-26
publish: true
tags:
  - windows
  - recovery
  - reset
  - troubleshooting
---
# Reset Windows PC - Reset Freezes or Fails

For when [[Reset Windows PC]] doesn't play: the reset from Settings fails, or the reset from Windows Recovery freezes partway through.

## Before you start

- **Get anything local off the machine first**: Downloads, Desktop, screenshots, anything not synced to OneDrive. A failed reset attempt can still wipe things. Check OneDrive is actually synced rather than assuming.
- Don't bother looking for a manufacturer-branded reset. Plenty of vendors (e.g. Lenovo, on some models) don't have one, and it's Windows Recovery either way.

## The fix: uninstall updates, then reset

If the reset from Windows Recovery freezes, and there are **no system restore points** to fall back on:

1. Boot into **Windows Recovery**: hold **Shift** while clicking Restart, or let it fail to boot 3 times
2. **Troubleshoot → Advanced options → Uninstall Updates**
3. Uninstall the **latest quality update** (and the latest feature update if that doesn't cut it)
4. Back to **Troubleshoot → Reset this PC**. It should now actually proceed.

A half-applied or broken pending update can block the reset. Removing it clears the way.

## If the reset works

It rules out basically everything except BIOS and hardware. If the original problem comes back after a clean reset, stop blaming Windows.

## After the reset

- Do the **Entra join / "Access work or school" sign-in with the user present**, using their own credentials, rather than a Temporary Access Pass. A TAP is single-use and short-lived, so it can leave the join half-finished.
- If the site also has **on-prem AD**, remember the machine may need joining to the domain instead of, or as well as, Entra.
- If a domain-joined machine throws a **trust relationship error**, disconnect it from the domain and rejoin rather than fighting it.

also... [[Crash Dump Analysis with WinDbg]] if the reset was to fix crashes or freezes. Check the dumps first, because it might just be a driver.
