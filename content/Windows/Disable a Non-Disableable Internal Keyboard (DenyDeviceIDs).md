---
category: work-note
tags:
  - windows
  - keyboard
  - registry
  - drivers
  - grouppolicy
  - hardware
date: 2026-09-18
publish: true
---
# Disable a Non-Disableable Internal Keyboard (DenyDeviceIDs)

Symptom: a laptop's internal keyboard has a physical fault (stuck/shorted key) and floods input continuously — including interleaving with an external keyboard plugged in as a workaround, since both devices send input concurrently. Normal disable methods refuse to touch the device.

Case this came from: [[Greenhood Clients/SMA/17-09-26 - Mitch Keyboard Fault]] — HP laptop, stuck A+V keys.

## Step 1 — identify the right device (easy to get wrong)

```powershell
Get-PnpDevice -Class Keyboard, HIDClass -PresentOnly | Format-Table FriendlyName, InstanceId, Status, Class -AutoSize
```

Watch for false leads in the list:
- **`ACPI\HPQ8002`-style entries** ("Standard 101/102-Key or Microsoft Natural PS/2 Keyboard for HP Hotkey Support") can be misread as a separate hotkey-only virtual device. On the affected model, this was actually the *only* keyboard-class node exposed for the internal keyboard — HP's hotkey filter driver wraps the whole interface rather than layering on top of a separate "real" keyboard device. Don't assume it's safe to ignore just because of the name.
- **I2C HID devices with a Synaptics ID (`SYNA****`)** are almost always the **touchpad**, not the keyboard. Disabling one of these will silently do nothing for a keyboard fault (and kill the trackpad).

## Step 2 — try the normal disable paths first

```powershell
Disable-PnpDevice -InstanceId "<InstanceId>" -Confirm:$false
```

If that fails with:
```
Disable-PnpDevice : Not supported
FullyQualifiedErrorId : HRESULT 0x8004100c,Disable-PnpDevice
```
...it means the device doesn't implement the CM_Disable_DevNode method the cmdlet relies on. `pnputil /disable-device` goes through the same underlying API and will fail the same way. Also try Device Manager GUI (right-click → Disable device) — it's a different code path and occasionally succeeds where the cmdlet doesn't. If Device Manager doesn't even show a Disable option for the device at all, it's flagged non-disableable at the driver/capability level and no runtime method will touch it.

## Step 3 — DenyDeviceIDs policy (works when runtime disable is refused)

This blocks the driver from loading at all, one layer below the disable/enable API — it doesn't care that the device reports itself as non-disableable.

1. Get the device's hardware IDs (not the instance ID):
```powershell
Get-PnpDeviceProperty -InstanceId "<InstanceId>" -KeyName DEVPKEY_Device_HardwareIds | Select -Expand Data
```
Typical output, most to least specific:
```
ACPI\VEN_HPQ&DEV_8002
ACPI\HPQ8002
*HPQ8002
```
Use the **bus-scoped** ID (`ACPI\HPQ8002` or the equivalent verbose form) — not the leading-`*` compatible ID. The `*` form has no bus prefix and matches that ID on *any* bus/enumerator, which is unnecessarily broad and could catch an unrelated device.

2. Set the policy:
```powershell
$regPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions"
New-Item -Path $regPath -Force | Out-Null
New-ItemProperty -Path $regPath -Name "DenyDeviceIDs" -PropertyType MultiString -Value @("ACPI\HPQ8002") -Force
New-ItemProperty -Path $regPath -Name "DenyDeviceIDsRetroactive" -PropertyType DWord -Value 1 -Force
gpupdate /force
Restart-Computer
```
`DenyDeviceIDsRetroactive` is the part that matters — without it, the policy only blocks *future* installs and leaves the already-loaded device running. Reboot is required for the retroactive eviction to actually happen.

Equivalent GUI path (same registry values, if you'd rather not touch regedit): Local Group Policy Editor → Computer Configuration → Administrative Templates → System → Device Installation → Device Installation Restrictions → "Prevent installation of devices that match any of these device IDs" (enter the hardware ID, tick "Also apply to matching devices that are already installed").

## Caveats

- This denies the **entire hardware ID**, not just the fault — if that ID also drives Fn-combo hotkeys (brightness/volume/etc.), those stop working too. Fine for a machine already moving to an external keyboard / being retired; not something to reach for on a keyboard you still need working normally.
- It's a **standing machine-wide policy**, not a one-shot action — it persists until the registry values are removed (and another reboot). Worth confirming the device really needs to stay dead before applying it.
- To reverse: delete the `DenyDeviceIDs` / `DenyDeviceIDsRetroactive` values (or the whole `Restrictions` key if nothing else uses it), `gpupdate /force`, reboot.
