---
category: tech
date: 2026-10-17
publish: true
tags:
  - kbdraft
  - publickb
  - 1password
  - passkey
  - entra
  - 2fa
---
# 1Password Passkeys for Entra Admin Accounts

Adding a passkey stored in 1Password to client M365 admin accounts, and what to do when Entra rejects it.

Related: [[1password 2FA Codes]], [[How to TAP in]]

## Standard step
1. Add the passkey to the admin account's 1Password entry.
2. **Sign out and test it.** Adding it isn't the same as it working.

## Error: "passkey does not meet the criteria set by your admin"

```
Your sign-in was successful but this passkey does not meet the criteria set by your admin. Try using another authentication method.
```

The tenant's Passkey (FIDO2) policy is restricting which authenticators are allowed.

**Check:** Entra admin centre > Protection > Authentication methods > Passkey (FIDO2) > Configure

1. **Key restrictions** (allow-list): if enforced, add the 1Password AAGUID:
   ```
   bada5566-a7aa-401f-bd96-45619a55120d
   ```
2. **Enforce attestation**: if it's on, only hardware-bound passkeys are accepted. 1Password passkeys are **synced**, so they fail attestation, and **the AAGUID alone won't fix it**.
3. The route when attestation must stay on: create a **separate passkey profile** for the admin accounts, without attestation, and leave the main policy alone.