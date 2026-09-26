---
category: how-to
date: 2026-09-26
publish: true
tags:
  - windows
  - bsod
  - windbg
  - drivers
  - troubleshooting
---
# Crash Dump Analysis with WinDbg

For when a PC is "running like hell" or blue-screening and Event Viewer is just a wall of noise. Stop guessing from symptoms and let WinDbg tell you which driver is actually at fault. It takes about 20 minutes and beats reinstalling random drivers.

## 1. Find the dumps

There are two kinds, and you want both:

| Type | Location | What it means |
|---|---|---|
| **BSOD minidumps** | `C:\Windows\Minidump\` | Real crashes. The PC blue-screened and rebooted. |
| **Live kernel dumps** | `C:\Windows\LiveKernelReports\` (e.g. the `WATCHDOG\` subfolder) | Non-fatal. Windows caught a driver misbehaving and snapshotted it *without* crashing. |

The live kernel ones are the sneaky part. They don't crash anything, but a pile of them shows up as a flood of "Information" error-reporting events with nothing else visibly wrong. That is often the real cause of "slow but no errors".

Also check Event Viewer for **Event ID 41** (unclean shutdown) with no matching dump. That's a hang hard enough that Windows couldn't write anything, often the same fault in a worse form.

Before trusting the dump history, confirm dumps are actually being written: `HKLM\SYSTEM\CurrentControlSet\Control\CrashControl` → `CrashDumpEnabled` should be non-zero (3 = small memory dump).

## 2. Install WinDbg

Get **WinDbg** from the Microsoft Store (or `winget install Microsoft.WinDbg`). Copy the `.dmp` files somewhere local first, because they're locked or admin-only in their original folders.

## 3. Run the analysis

1. WinDbg → **File → Open dump file** → pick the `.dmp`
2. If symbols don't load on their own, set the symbol path:
   ```
   .sympath srv*C:\Symbols*https://msdl.microsoft.com/download/symbols
   .reload
   ```
3. Run:
   ```
   !analyze -v
   ```
4. Wait. The first run downloads symbols and takes a few minutes.

## 4. What to read in the output

Skip the rest and look at these lines:

- **`BUGCHECK_CODE`**: the stop code (e.g. `9f`, `193`)
- **`IMAGE_NAME` / `MODULE_NAME`**: the driver the crash landed in
- **`PROCESS_NAME`**: what was running at the time
- **`FAILURE_BUCKET_ID`**: the fingerprint. **This is the important one.**

**Compare `FAILURE_BUCKET_ID` across several dumps**, at least the earliest and the latest. If it's identical every time over weeks or months, you've got one specific, reproducible driver bug, not general flakiness. If it's all over the place, think RAM, heat, or power instead.

## 5. Common patterns

| Bucket / code | Meaning | Usual fix |
|---|---|---|
| `0x9F_3_POWER_DOWN_ndis!...` | `DRIVER_POWER_STATE_FAILURE`. The network stack was waiting on the **Wi-Fi driver** to finish powering down during sleep and it never answered. | Update the Wi-Fi driver. If it's already current: Device Manager → network adapter → Power Management → untick **"Allow the computer to turn off this device to save power"** |
| `LKD_0x193_dxgkrnl!...` in `dwm.exe` | Live dump (non-fatal) from the **graphics kernel driver**. Shows up as recurring freezes and stutters, not crashes. | Update the GPU driver (e.g. Intel `igdkmdn64.sys`). Check the driver date, because a year-old one is a strong suspect. |

A single machine can have **more than one** of these at once, e.g. graphics behind the freezes and Wi-Fi behind the BSODs. Treat each bucket as its own problem.

## 6. Rule out the boring stuff

- **RAM**: check whether Windows Memory Diagnostic has already run (it often auto-runs after crashes). If it has passed clean, move on.
- **BIOS**: check the installed version against the vendor's latest. If the installed one is *newer* than what's published, anti-rollback will block the update anyway, so don't chase it.
- **OEM utilities** (Vantage, etc.) crashing on their own are a side issue. Uninstall them and move on.

## 7. After the fix

Check the dump folders again in a couple of weeks. No new dumps with the same bucket ID means it worked.

also... [[Keyboard Code 19 - Orphaned Filter Driver]] for another "it's a driver, not hardware" one.
