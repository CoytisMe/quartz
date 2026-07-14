---
category: tech
date: 2026-07-06
publish: true
tags:
  - homelab
  - proxmox
  - gpu-passthrough
---

## This is defunct but nice to have for later
I wanted to be able to stream Jellyfin to Discord via the Windows VM, I found another way with a laptop and Ubuntu.

I may still need the GPU for transcoding later but right now who can be fucked>
## Goal
Pass the Zotac GTX 1080Ti (currently unused, no passthrough configured — see [[coytishost hardware reference]]) through to WinVMGeneral (192.168.1.91) so it can hardware decode/encode video. Fixes two problems:
- Jellyfin → Discord streaming chugging hard (all-software decode/encode on CPU)
- NDI Studio Monitor refusing to load (no GPU available to the VM at all)

## Why this works well here
- CPU is i9-9900KF — no iGPU, so Proxmox host has never used the 1080Ti for its own display and runs fully headless already. No "host needs the GPU too" conflict.
- CPU + Z390 chipset both support VT-d/IOMMU, which passthrough requires.
- Only one VM needs the card at a time (Windows), so straight exclusive PCI passthrough is the right approach — no need for vGPU/sharing hacks.

## Caveat
Passthrough is exclusive — while WinVMGeneral is running and holding the card, neither the Proxmox host nor the Ubuntu VM can use it. Not an issue here since only Windows needs it.

## Steps

1. **Enable VT-d in BIOS** (MSI Z390M) if not already on.
2. **Enable IOMMU on the Proxmox host** — add `intel_iommu=on iommu=pt` to the kernel cmdline / grub config, `update-grub`, reboot.
3. **Bind the GPU to vfio-pci**:
   - `lspci -nn | grep -i nvidia` to get the PCI IDs for both GPU functions (VGA + HDMI audio).
   - Blacklist `nouveau` in `/etc/modprobe.d/blacklist.conf`.
   - Add `options vfio-pci ids=10de:xxxx,10de:yyyy` in `/etc/modprobe.d/vfio.conf`.
   - `update-initramfs -u -k all`, reboot.
   - Verify with `lspci -k -s <busid>` — driver in use should show `vfio-pci`.
4. **Check WinVMGeneral's current config first** before changing boot mode — passthrough works best with:
   - Machine type: **q35**
   - BIOS: **OVMF (UEFI)**
   - Changing this on an already-installed Windows VM needs care (may need an EFI disk added / boot repair).
5. **Add the GPU as a PCI Device** in WinVMGeneral's hardware — tick "All Functions" and "PCI-Express".
6. **Hide KVM from the guest** — Proxmox → VM → Options → "Hide KVM hardware virtualization". Without this, consumer GeForce drivers throw **Code 43** in Device Manager when they detect they're in a VM.
7. Boot the VM, install normal NVIDIA drivers — GPU should appear as a real second adapter. Test Jellyfin→Discord stream and NDI Studio Monitor.
