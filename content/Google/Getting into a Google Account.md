---
category: tech
date: 2026-06-29
publish: true
tags:
  - google-workspace
  - google-admin
  - 2fa
  - backup-codes
  - account-access
---
## Different Philosophies

This isn't Microsoft 365, you can't TAP in a Google account, if you don't know the password you need to change it.

The way I like to think about it is that Microsoft is more admin focused — the user's account is never really theirs. Google is more user focused — admins can't just drop in.

also... [[How to TAP in]]

## Keep in mind

This is not quiet. The user will get a 2FA notification when you try to sign in, and they will get a 'Was this you' email when you succeed. This usually isn't an issue as you typically do this after hours.

But if a manager wants to have a snoop, let them know this is **NOT** quiet like it is with Microsoft.

also... [[Detected Unusual Signin]]

### Step One — Google Admin

1. Find your user
2. Reset their password if you don't have it
3. Go to the Security tab
4. Click on 2-step verification
5. 'Get Backup Verification Codes'
6. Record a code or two

![[Tasks-4.png]]

### Step 2 — Sign In

**Gmail, or any Google service**

1. Sign in as the user
2. It will ask for their 2FA
3. Hit 'Try another way'
4. Hit 'Use a Backup Code'
5. Use a backup code

![[Tasks-5.png]]
