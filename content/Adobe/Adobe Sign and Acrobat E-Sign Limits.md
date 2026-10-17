---
category: tech
date: 2026-10-17
publish: true
tags:
  - kbdraft
  - publickb
  - adobe
  - adobesign
  - esign
  - licensing
source:
---
# How many signs per Adobe?
## They make it surprisingly hard to figure out
### Cause they're cunts

Limits that come up when a client uses Acrobat/Adobe Sign for e-signing, or wants to swap Docusign for it.

## Transaction cap: 150 per licence per year
- Acrobat Standard/Pro on **business, team and enterprise** plans caps e-sign at **150 transactions per licence per year**.
- For higher volume, go through Adobe's **VIP/Enterprise sales channel**.
- Before a client cancels Docusign, check last year's envelope count against the cap.
- In the case seen, the client still went with Acrobat Standard for Teams over Docusign.

## File size: ~10 MB send limit
- Adobe Sign won't send a document over about **10 MB**.
- A document that's already signed and locked can't be compressed or have pages pulled out, so you can't shrink it to fit.
- Adobe support can raise the send limit **without** an Enterprise licence upgrade, so contact them before upselling a licence.

## Setup tip: use an alias as the admin account
When setting up Acrobat for Teams, create the Adobe admin account on a **mailbox alias** (e.g. `adobe@<domain>`) rather than a person's address, so ownership can be moved later. Store its login in the password manager.

## Gaps / to confirm
- [ ] Check the 150 cap against Adobe's current plan docs and add a link.
- [ ] Did Adobe support actually raise the send limit in the 10 MB case, and how long did it take?

Related: [[Adobe PDF Locked Form Fields]], [[Adobe Acrobat Clean Reinstall]]
