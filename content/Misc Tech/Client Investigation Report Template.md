---
category: work-note
date: 2026-10-17
publish: false
tags:
  - kbdraft
  - greenhoodkb
  - report
  - security
  - audit
  - procedures
---
# Client Investigation Report Template

My Boss told me never to use AI to write communications to clients, I never did.

He asked me to.

The structure for a client-facing report after a security sweep, an evidence request or an "did X happen?" investigation. Built from the SMA Axcelerate API key audit (Oct 2026), with the Trucare laptop activity report (Apr 2026) as the earlier version of the same kind of job.

Probably lives in [[Greenhood/Procedures]] once finalised.

## Rules
- Keep it to about two pages. Plain English. The reader is the client contact, not IT.
- **Never put the secret itself in the report.** Identify it by its first segment only (e.g. "the key beginning `027B...`").
- Say up front what a log entry being absent does and doesn't mean (Trucare: "we don't have activity tracking; the absence of a log entry isn't proof of no activity").
- Mark anything to check before sending as ==highlighted==, and put a "draft, not sent" callout at the top until it goes.

## Structure

```markdown
# <Subject>: <Type of audit/report>

**Prepared for:** <client contact>, <client>
**Prepared by:** Greenhood IT
**Date:** <date>
**Ticket:** <link>

## Summary
What we were asked, and the one-line answer (bold). Then the main caveat in a sentence.

## What we did
### 1. <Source/tool> (e.g. Purview Audit, Ninja activity, Google admin logs)
- What it records, in one line
- Date range searched, and why that range (retention limits)
- What we searched for
**Result:** ...

### 2. <Next source>
...

### 3. Manual review
What was exported and checked by hand/offline, and what the hits turned out to be.

## Limitations
- Retention windows
- Content that couldn't be searched (partially indexed items, images)
- What was out of scope (other platforms, personal devices, vendor systems)

## Recommendations
Numbered. Lead with the action that closes the risk regardless of the findings (e.g. rotate the key), then the evidence the vendor holds, then hygiene and licensing.

## Conclusion
Two or three sentences restating the finding and the top recommendation.

Greenhood IT
```

