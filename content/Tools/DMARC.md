---
category: work-note
date: 2026-06-03
publish: true
tags:
  - dmarc
  - email
  - dns
  - security
---

# DMARC

## What is DMARC?

DMARC (Domain-based Message Authentication, Reporting & Conformance) is a DNS TXT record that tells receiving mail servers what to do when an email claims to be from your domain but fails authentication checks (SPF and/or DKIM).

It builds on top of:
- **SPF** — defines which mail servers are allowed to send on behalf of your domain
- **DKIM** — cryptographically signs outgoing mail so receivers can verify it wasn't tampered with

DMARC ties these together and adds a **policy** (what to do on failure) and **reporting** (where to send failure data).

---

## The DMARC Record

Published as a DNS TXT record at `_dmarc.yourdomain.com`.

### Key tags

| Tag | Meaning |
|-----|---------|
| `v=DMARC1` | Version — always required, always this value |
| `p=` | Policy — what to do with failing mail (see below) |
| `rua=` | Aggregate report destination — where to send daily XML summary reports |
| `ruf=` | Forensic report destination — per-failure reports (less common, privacy implications) |
| `pct=` | Percentage of mail the policy applies to (default 100) |
| `sp=` | Subdomain policy (inherits `p=` if omitted) |

---

## Policy Options (`p=`)

### `p=none`
- **Monitor only** — do nothing, just report
- Failing mail is still delivered as normal
- Safe starting point — won't break anything
- Useful for understanding your mail flows before enforcing

### `p=quarantine`
- Failing mail goes to the recipient's spam/junk folder
- Moderate enforcement — reduces impact of spoofing without hard-blocking

### `p=reject`
- Failing mail is rejected outright at the server level — never delivered
- Full protection against spoofing/phishing using your domain
- Only use once you're confident SPF and DKIM are solid, or you'll drop legit mail

---

## The Two Records in Question

### Cloudflare auto-generated
```
v=DMARC1; p=none; rua=mailto:958d64bc3f4b4124bc98677297c19bb1@dmarc-reports.cloudflare.net
```
- Policy: `none` (monitor only, no enforcement)
- Reporting: sends aggregate reports to Cloudflare's DMARC reporting service
- Cloudflare parses these and surfaces them in the Email Security dashboard
- **Good** — you get visibility into who's sending mail as your domain

### Boss's preference
```
v=DMARC1; p=none
```
- Policy: `none` (same — monitor only)
- No `rua=` tag — **no reports sent anywhere**
- Technically valid, meets the "we have DMARC" checkbox
- **Downside:** you're flying blind — no data on authentication failures or spoofing attempts

---

## Recommendation

Cloudflare's version is strictly better than bare `p=none` — same policy, but you actually get the reporting data. There's no downside to keeping the `rua=` tag.

Long-term goal for any domain should be moving toward `p=reject` once SPF and DKIM are verified working, to actually prevent spoofing.

**Suggested progression:**
1. `p=none` + `rua=` (monitor, collect data) ← where coytis.me is now
2. `p=quarantine; pct=10` (soft rollout)
3. `p=quarantine` (full quarantine)
4. `p=reject` (full enforcement)

---

## Tools

- [MXToolbox DMARC Checker](https://mxtoolbox.com/dmarc.aspx)
- [Cloudflare Email Security Dashboard](https://dash.cloudflare.com) → domain → Email → DMARC Management
- [DMARC Analyser](https://www.dmarcanalyzer.com/dmarc/checker/)
