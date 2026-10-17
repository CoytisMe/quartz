---
category: reference
date: 2026-10-06
publish: true
tags:
  - googleworkspace
  - licensing
  - googlebusinessprofile
---
# Google Workspace Licensing

Which Google Workspace tier does what, roughly what it costs in Australia, and the gotchas that bite small clients. Prices are AUD per user per month, checked October 2026. Google changes these, so confirm on the [pricing page](https://workspace.google.com/intl/en_au/pricing) before quoting.

## The tiers

| Plan | Cost (AUD) | Custom-domain Gmail | Storage | Meet | Notable extras |
|---|---|---|---|---|---|
| **Essentials Starter** | Free | ❌ No Gmail at all | 15 GB per user | Yes (time-limited) | Drive, Docs/Sheets/Slides, Meet, Chat, Calendar. Up to 100 users. You sign up with an existing work email address |
| **Cloud Identity Free** | Free | ❌ | — | — | An identity only (sign-in, SSO, device basics). See [[Google Workspace Cloud Identity]] |
| **Business Starter** | ~$9.90 annual, ~$11.80 flexible (monthly) | ✅ | 30 GB pooled per user | 100 participants, no recording | Gemini in Gmail only, basic security |
| **Business Standard** | ~$19.80 annual, ~$23.60 flexible | ✅ | 2 TB pooled per user | 150, with recording | Gemini across Docs/Meet etc., appointment booking, eSignature, migration tools |
| **Business Plus** | ~$30.90 annual, ~$37 flexible | ✅ | 5 TB pooled per user | 500, with attendance tracking | **Vault** (retention/eDiscovery), advanced endpoint management, stronger security |
| **Enterprise** | Quote only | ✅ | 5 TB+ | 1,000 | DLP, S/MIME, context-aware access, data regions |

Business plans are capped at 300 users. "Annual" means a one-year commitment; "flexible" means month to month, cancel anytime, at roughly 16–20% more.

## What you can and can't do on each

**Essentials Starter (free):**
- ✅ Collaborate in Drive/Docs and run Meet calls, using a team's existing (non-Google) email addresses
- ❌ No Gmail or custom-domain email. It's for teams whose mail lives somewhere else (e.g. M365)
- ❌ **Additional Google services** (YouTube, Blogger, Google Business Profile, Search and Assistant, Brand Accounts, etc.) are controlled by an admin setting that **email-verified Essentials accounts can't use**. Google's own docs say you have to verify the domain to unlock those controls ([source](https://knowledge.workspace.google.com/admin/users/advanced/turn-on-or-off-additional-google-services)). In practice, that's why a free Essentials account can't see its own Google Business Profile ([[05-10-26 - Business Profile Access]]).
- ❌ Limited admin controls, no Vault, no email security settings

**Business Starter:**
- ✅ Custom-domain Gmail, shared calendars, basic admin console
- ✅ Admin can turn on additional services (Business Profile, YouTube, etc.) per OU
- ❌ No Meet recording, no Vault, no eSignature; small storage pool fills up fast with a shared Drive
- Good default for a small office that just needs email

**Business Standard:**
- ✅ Everything a typical office actually asks for: recordings, 2 TB, eSignature, booking pages
- Usually the right tier once a client starts complaining about storage

**Business Plus:**
- ✅ Vault, so retention/litigation hold and eDiscovery. Needed for anyone with record-keeping obligations (financial advisers, legal)
- ✅ Advanced endpoint management (remote wipe, device approvals)

## Free alternatives for "we only need X"

| Need | Cheapest option |
|---|---|
| Just a Google Business Profile / Maps listing | A free **@gmail.com** account (consumer accounts can manage Business Profiles), ideally a shared one owned by the business, not a staff member |
| Just Docs/Drive access for someone without a licence | Cloud Identity Free ([[Google Workspace Cloud Identity]]) or Essentials Starter |
| Email on the company domain | Business Starter is the floor |

Before moving a Business Profile from a paid Workspace user to a free Gmail account, **add the Gmail account as an Owner on the profile first**, confirm it shows, then remove the Workspace user. Transferring ownership keeps the reviews, photos and history. Deleting the profile and re-creating it loses them.

## Related
- [[Google Workspace Cloud Identity]]
- [[Migrating Between Workspace Tenancies]]

## Sources
- [Google Workspace pricing (AU)](https://workspace.google.com/intl/en_au/pricing)
- [Google Workspace Essentials](https://workspace.google.com/essentials/)
- [Turn additional Google services on or off](https://knowledge.workspace.google.com/admin/users/advanced/turn-on-or-off-additional-google-services)
