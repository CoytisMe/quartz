---
category: how-to
tags:
  - windows
  - heic
  - images
  - apple
publish: true
---
## Introh
Not a lot to say, trying to find a fucking HEIC viewer that wasn't heinous

## Best Solution — Microsoft HEIF Image Extensions

Install the codec from the Microsoft Store. Once installed, HEIC files open natively in the Windows **Photos app** — double click to open, arrow keys to browse. No extra app, no conversion needed.

**Microsoft Store link:** https://apps.microsoft.com/detail/9pmmsr1cgpwg?hl=en-GB&gl=AU

### Free vs Paid
- The standard Microsoft version costs **$0.99**
- A free **OEM/Device Manufacturer** version may be available depending on your device — check the Store listing first before paying

## Also Required — HEVC Video Extensions

Most modern iPhone photos use HEVC compression, so the HEIF codec alone isn't enough. You'll also need the HEVC Video Extensions from the Store.

Search **"HEVC Video Extensions"** in the Microsoft Store — same deal as above, check for a free device manufacturer version first before paying (~$1.49 for the paid version).

## Alternative — ImageGlass (Recommended Standalone Viewer)

ImageGlass is a clean, open-source image viewer that handles HEIC well. Free Classic version available — the MS Store version is $15.

**Download (Classic, free):** https://imageglass.org/release/imageglass-9-4-1-15-61

**Still requires the HEIF/HEVC codecs above to be installed.**

### Setting as Default
ImageGlass won't appear in the default apps list automatically — you need to set it manually:
1. Right-click a HEIC file → **Open with** → **Choose another app**
2. Scroll down and click **More apps** or **Look for another app on this PC**
3. Navigate to `C:\Program Files\ImageGlass\` and select the `.exe`

## Fallback — CopyTrans Viewer

If the codec route doesn't work, CopyTrans Viewer is the community favourite standalone app. Free, clean interface, arrow navigation, no conversion required.

https://copytrans.studio/copytrans-viewer/

## Notes

- HEIC is Apple's image format (iPhone default since iOS 11)
- Windows has no native HEIC support without the codecs installed
- Both HEIF and HEVC extensions are needed for most iPhone photos
- Check if you already own the extensions on another PC before purchasing — licences are tied to your Microsoft account
