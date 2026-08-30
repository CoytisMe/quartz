---
category: tech
tags:
  - microsoft
  - office
  - mac
  - troubleshooting
date: 2026-08-06
publish: true
---
# MS Office for Mac — Sign-in Error

**Error:**
"Another account from your organisation is already signed in on this device. Try again with a different account."

**Solution (via Google search):**

### One:
- Close all Office Programs
- Run in Terminal:
```bash
defaults write com.microsoft.Word ResetOneAuthCreds -bool YES
```

## Result
Didn't work, the MacOS was 11.7.11 and that's too old for auth.
## Followup

This resets the OneAuth credential cache for Word specifically — if the same error shows up in Excel/PowerPoint/Outlook, the same command pattern likely applies with `com.microsoft.Excel`, `com.microsoft.Powerpoint`, `com.microsoft.Outlook` etc. Worth noting whether "One" implies there was a second fix option that didn't make it into the daily note — if the first one doesn't stick, check back with Rick for what "Two" was supposed to be.

## Update 24-08-26

Same underlying cause confirmed: can't log in because another user from the same org is already signed in on the device. Next thing to try — the official [Microsoft Office Removal Tool for Mac](https://support.microsoft.com/en-us/microsoft-365-activation-licensing/how-to-remove-office-license-files-on-a-mac), which clears license files more thoroughly than the OneAuth cache reset above. Given the OneAuth reset already failed once (old macOS, 11.7.11), this is worth trying next — full removal rather than a targeted cache clear.

Source: [[Dailys/2026/08/24-08-26]]
