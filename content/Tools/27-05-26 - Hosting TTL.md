---
category: work-note
tags:
  - hosting
  - dns
  - microsoft
  - domain-validation
date: 2026-05-27
source: "[[Dailys/2026/05/27-05-26]]"
---
# Hosting — Domain Validation TTL Issue

TTL too long when trying to authenticate domain with Microsoft.

**Paddy's Response:**
Normally I would use the TTL manager and WHM to reduce the overall TTL for the domain down to 3600 or even 900 while we do validation. On our VPS you can't mix and match TTL in the one domain zone. In this case the cPanel account is not on our VPS. It is with Clint at the Artistree — he is our guy for this going forwards. So you will need to put up with a longer TTL or email him and ask him to reduce it down to 900 for the zone at the WHM level.

## Followup

**Action outstanding:** Email Clint at Artistree and ask him to reduce the TTL to 900 for the zone at the WHM level. Once validation is complete, remind him to set it back.

The key constraint here is that their VPS doesn't allow mixed TTLs in a single zone — so the fix has to happen at Clint's end, not Paddy's. Clint is now the primary contact for any hosting work on this account going forwards — worth saving his contact details if not already done.

Microsoft domain validation TXT records typically propagate within minutes at TTL 900, so requesting the reduction is well worth doing before the next attempt.
