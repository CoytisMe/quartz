---
category: research
date: 2026-06-01
tags:
  - freshdesk
  - api
  - claude
  - privacy
  - security
  - msp
publish: true
---
# My Boss Was Curious
Apparently their 'considerations' to be 'Considered' when pumping all of your clients data through a machine. So I sent him this.

Make me laugh as I've just asked Claude if it can be trusted and used it's answer as evidence, not sure if that's conceptually a good thing.

## Freshdesk API + Claude — Privacy & Data Security

Notes from evaluating whether to give Claude API key access to the Freshdesk service board.

---

### What Actually Happens With the Data

When Claude pulls tickets via the API, that content — ticket descriptions, contact details, client names, any sensitive info in the ticket body — comes into the context window and flows through Anthropic's infrastructure. For an MSP this means potentially client infrastructure details, passwords pasted into tickets, network info, etc.

---

### Anthropic's Data Handling (Claude Code / API)

Claude Code runs on the API, not the consumer Claude.ai product. Key points:

- Anthropic does **not train on API conversations** by default for paid users
- There may still be some retention for safety/trust & safety review purposes
- Verify current terms at [anthropic.com/privacy](https://anthropic.com/privacy) — it changes

---

### API Key Security

- Must be stored as an **environment variable** — never hardcoded in a file or pasted into conversation
- A leaked key = full Freshdesk account access
- Freshdesk supports scoped API keys on some plans but it's typically all-or-nothing

---

### Bigger Concern for MSP Context

Tickets likely contain data belonging to **clients**, not just internal data. Depending on client agreements, routing their data through a third-party AI may be a grey area or a problem — even on a personal paid account.

---

### Practical Verdict

| Use case | Risk level |
|---|---|
| Internal tickets, metadata queries, stats | Low |
| Pulling full ticket bodies for triage/summaries | Medium |
| Tickets containing client credentials, PII, infrastructure details | Higher — avoid or be selective |

Treat it like any other third-party SaaS integration — not zero risk, but manageable with care.
