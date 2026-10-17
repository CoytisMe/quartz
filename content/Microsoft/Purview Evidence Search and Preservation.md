---
category: tech
date: 2026-10-17
publish: true
tags:
  - kbdraft
  - publickb
  - purview
  - ediscovery
  - audit
  - m365
  - security
---
# Lol I had to search for an API Key

Two jobs that come up after an incident or an awkward departure:

1. **Preserve** a user's mailbox/files (hold, export, hash).
2. **Search** the whole tenant for something that shouldn't be there, e.g. a leaked API key.

Related: [[M365 Unified Audit Log]], [[SHA256 File Hash Export]]

## Licensing
- eDiscovery (Standard) cases and litigation hold need **Business Premium** or Exchange Online Plan 2. Business Standard doesn't include them.
- In the case seen, a Business Premium licence had to be added before the case and hold would work.
- Don't drop the licence while the hold is still needed.

## Audit log vs eDiscovery

| | Audit (Purview > Audit) | eDiscovery (Purview > eDiscovery) |
|---|---|---|
| Searches | Activity: sign-ins, file access, mailbox and admin actions | Content: mail, files, Teams chats |
| Retention | **180 days** on Business Premium/E3. Anything older is gone | Whatever's still in the tenant |
| Speed | Quick | The search is quick, the **export takes hours** |

If the incident timeline is older than 180 days, the audit log can't help. Say so up front.

## Preserving a user (hold + export + hash)
1. Confirm audit logging is on (see [[M365 Unified Audit Log]]).
2. Purview > eDiscovery > **Create case**, using a naming convention like `<client>-<mm-yy>-Pres`.
3. Add a **hold policy** (`<client>-<mm-yy>-Hold`) and wait for the status to show **On Hold**.
4. Confirm the litigation hold in PowerShell.
5. Search and export. Refresh the page if the export looks stuck.
6. Hash the export with [[SHA256 File Hash Export]]. Hash from a **local, non-synced** copy: OneDrive/Drive sync holding the results CSV open causes an "in use" error.
7. Pull the user's sign-in logs as well, if relevant.
8. Store everything in an audits folder and record the chain of custody.

## Searching for a leaked secret (API key, password)
1. **Audit search** over the maximum window: full key, then the first half, the first segment, and the first segment with a wildcard. Try any API-related record type too (`DataInsightsRestAPI` was the only one available).
2. **eDiscovery case** with the sources set to **all people and groups** plus **all public folders**. That covers current, departed, guest and unlicensed users and shared channels.
3. Query: the full key **OR** each segment on its own, so a copy split across lines or with the dashes stripped still matches.
4. **Short segments (4 characters) flood the results.** They collide with random GUIDs in Teams call IDs, image references, spam headers and page caches. Search only the full key and segments of 5+ characters.
5. Export the hits and scan them offline for the key with and without dashes, in both cases and both text encodings. In the case seen, all 254 hits were coincidental GUID fragments.

## Limitations to state in any report
- Audit retention (180 days)
- Partially indexed items (encrypted, large, odd file types) aren't searched
- Images and screenshots aren't read for text
- Nothing outside M365 is covered: the vendor platform, the website and hosting, repos, personal devices