---
category: tech
date: 2026-08-06
publish: true
tags:
  - microsoft365
  - exchange
  - security
  - defender
  - compliance
---

# Outbound Spam Restriction — Investigation Playbook

## When This Fires

Microsoft 365 auto-restricts a mailbox from sending external mail when its abuse detection sees a spike in outbound volume that looks like a spray/spam campaign — e.g. a Defender alert like "User restricted from sending email" / "Potentially compromised user account". It usually means one of two things:

1. **Actual compromise** — credentials phished, mailbox used to blast spam/phishing.
2. **Legitimate bulk send** — a newsletter, event invite, or mail-merge sent directly from a live mailbox instead of a proper mailing platform, tripping the same volume heuristics.

Don't assume either way from the client's explanation alone — confirm with message trace and sign-in logs before lifting the restriction.

## Investigation Steps

1. **Message trace** — Defender portal (or Exchange admin center) > Mail flow > Message trace. Search the affected mailbox for the alert window.
   - Spam/phishing pattern: random unrelated domains, generic/urgent subject lines, high bounce rate.
   - Legit bulk send: recognisable, relevant recipient list (customers, contacts, members) and a normal marketing subject line.
2. **Sign-in logs** (Entra) — check for unfamiliar country/IP or impossible travel around the same time. See [[M365 Entra Diagnostic Logging]].
3. **Inbox rules** — check the mailbox for hidden rules (auto-delete/forward on replies) that attackers use to hide a spam run from the real owner.
4. **MFA / app registrations** — check for newly registered MFA methods or OAuth app consents that shouldn't be there. See [[App Consent]].
5. **Decide:**
   - Signs of compromise → reset password, revoke all sessions/tokens, remove rogue rules/app grants, *then* release the restriction.
   - Confirmed legitimate → release the restriction, and see below to stop it recurring.

## Releasing the Restriction

Defender portal > **Email & collaboration** > **Review** > **Restricted entities** > select the user > **Unblock**.

> Tip: the left nav shows the child pages of a section once it's expanded — you don't need to land on the "Review" landing page first. Expand **Email & collaboration → Review** in the sidebar and go straight to **Restricted entities**, rather than clicking through the parent page the way most guides describe it.

## Stopping It Recurring — Adjust the Outbound Sending Thresholds

The trigger is the tenant's **outbound spam filter policy** — it caps how many recipients a single mailbox can send to per hour/day before auto-restricting. Defaults are tuned for normal user behaviour, not bulk sends.

**UI path:** Defender portal > **Email & collaboration** > **Policies & rules** > **Threat policies** > **Anti-spam** > **Anti-spam outbound policy (Default)** > Edit protection settings > **Message limits**.

**Default limits** (verify current values in the tenant — Microsoft has adjusted these over time):

| Setting | Default |
|---|---|
| External recipients per hour | 500 |
| Internal recipients per hour | 1000 |
| Recipients per day | 1000 |
| Action when threshold reached | Restrict user from sending |

**PowerShell (ExchangeOnlineManagement module):**
```powershell
Connect-ExchangeOnline

# Check current values
Get-HostedOutboundSpamFilterPolicy | Select-Object Name,RecipientLimitExternalPerHour,RecipientLimitInternalPerHour,RecipientLimitPerDay,ActionWhenThresholdReached
```

```powershell
# Raise the ceiling for a tenant with a known, recurring legitimate bulk sender
Set-HostedOutboundSpamFilterPolicy -Identity Default -RecipientLimitExternalPerHour 750 -RecipientLimitPerDay 2000
```

Only raise this once you've confirmed the volume is legitimate — it's the same lever that catches a real compromised account, so loosening it tenant-wide trades detection sensitivity for convenience.

## Better Fix: Get Bulk Sends Off the Mailbox Entirely

Recurring newsletters/campaigns shouldn't go out from a live user or shared mailbox at all — that's what trips this every time, and it also burns the domain's sender reputation for genuine mail. Move it to a proper ESP instead:

- **Mailchimp** / **Constant Contact** — easiest for non-technical staff, built-in templates and list management.
- **Brevo** (ex-Sendinblue) — cheaper at volume, still has a simple editor.
- **SendGrid** / **Postmark** — better for transactional-style or dev-driven sends.

All of these handle unsubscribe/compliance (Spam Act/CAN-SPAM), bounce and complaint handling, and send from their own warmed-up infrastructure (or a subdomain of the client's domain via SPF/DKIM delegation) — so the client's actual mailbox and domain reputation stay clean no matter how big the list gets.

## Notes

- This is the same alerting path that catches genuine account compromise — treat every restriction as unconfirmed until message trace backs up the explanation, even when the client's story sounds plausible.
- Once a client says "we send newsletters," that's the cue to get them off the mailbox and onto an ESP, not just to raise the tenant threshold.
