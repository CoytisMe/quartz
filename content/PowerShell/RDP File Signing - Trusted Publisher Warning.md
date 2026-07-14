---
category: reference
date: 2026-07-05
publish: false
tags:
  - rdp
  - code-signing
  - powershell
  - certificates
  - command-palette
---

# RDP File Signing — Trusted Publisher Warning

## Goal
Launching `C:\Users\Rick\Documents\RDPs\WinVMGeneral.rdp` via the PowerToys [[SSH Config and Command Palette]] kept popping a "do you trust this publisher?" confirmation prompt every time. Tried to get rid of it by code-signing the `.rdp` file with a self-signed certificate.

## What was tried

```powershell
PS C:\Users\Rick> $cert = New-SelfSignedCertificate -Type CodeSigningCert -Subject "CN=YourCompany" -KeyUsage DigitalSignature -CertStoreLocation "Cert:\LocalMachine\My"
PS C:\Users\Rick> Get-ChildItem Cert:\currentuser\My -CodeSigningCert
PS C:\Users\Rick> Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.EnhancedKeyUsageList -match "Code Signing" }

   PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\My

Thumbprint                                Subject              EnhancedKeyUsageList
----------                                -------              --------------------
3B1A68EDF17A097CF3C5119D4ECE204523EC8D92  CN=YourCompany       Code Signing

PS C:\Users\Rick> rdpsign.exe /sha256 3B1A68EDF17A097CF3C5119D4ECE204523EC8D92 "C:\Users\Rick\Documents\RDPs\WinVMGeneral.rdp"
All rdp file(s) have been succesfully signed.
PS C:\Users\Rick>
```

The signing itself worked — `rdpsign.exe` confirmed success — but the warning **still showed up** when launching the file.

## Response — why it didn't work

`rdpsign.exe` only *signs* the file; it doesn't make Windows *trust* the signer. The Remote Desktop Connection client checks the signing certificate against two separate trust mechanisms, and a fresh self-signed cert satisfies neither by default:

1. **Certificate trust chain** — the cert needs to sit in `Cert:\LocalMachine\Root` (Trusted Root Certification Authorities) or `Cert:\LocalMachine\TrustedPeople`, not just `Cert:\LocalMachine\My` (Personal). `My` is where you *generate/hold* certs, not where Windows looks for trust decisions.
2. **RDP-specific publisher allowlist** — even a chain-trusted cert isn't enough for `.rdp` files specifically. Windows also checks a Group Policy / registry setting that whitelists trusted RDP publishers by thumbprint:
   - `Computer Configuration → Administrative Templates → Windows Components → Remote Desktop Services → Remote Desktop Connection Client → Specify SHA1 thumbprints of certificates representing trusted .rdp publishers`
   - Or via registry: `HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services\TrustedCertThumbprints` (REG_MULTI_SZ, one thumbprint per line).
   - Note this GPO setting wants the **SHA1** thumbprint, while the signing command above used `/sha256` — worth double-checking `rdpsign.exe` and the GPO are pointed at matching thumbprint formats/algorithms, or the mismatch alone would explain the prompt persisting.

So the file was validly signed, but Windows had no reason to trust *that* signer for *that* purpose — the prompt is doing exactly what it's designed to do until both of the above are satisfied.

## Followup
- [ ] Import the cert into `Cert:\LocalMachine\Root` (or `TrustedPeople`) so the chain resolves: `Import-Certificate -FilePath <exported .cer> -CertStoreLocation Cert:\LocalMachine\Root`
- [ ] Add the cert's thumbprint to the "trusted .rdp publishers" GPO (`gpedit.msc`) or the `TrustedCertThumbprints` registry value directly
- [ ] Confirm whether `rdpsign.exe /sha256` and the GPO thumbprint field expect the same hash — get the SHA1 thumbprint if the policy needs it, re-sign if necessary
- [ ] Re-test launching `WinVMGeneral.rdp` from the command palette to confirm the prompt is gone
- [ ] If this works, consider whether it's worth doing for other frequently-launched `.rdp` files (e.g. anything else tied to [[GPU Passthrough to Windows VM]]'s WinVMGeneral box)
