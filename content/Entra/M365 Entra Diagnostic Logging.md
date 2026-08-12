---
category: how-to
date: 2026-06-05
publish: true
tags:
  - microsoft365
  - entra
  - audit
  - compliance
  - security
---

# M365 Entra Diagnostic Logging

See also: [[M365 Unified Audit Log]]
## What This Is

Entra (Azure AD) has sign-in and audit logs that are **separate from the Unified Audit Log**. They're visible in the Entra portal for only **7 days** by default — after that they're gone unless you ship them somewhere.

The key logs UAL doesn't cover well:

| Log                            | What it captures                                        |
| ------------------------------ | ------------------------------------------------------- |
| `SignInLogs`                   | Interactive user sign-ins                               |
| `NonInteractiveUserSignInLogs` | OAuth token refreshes, app sign-ins on behalf of a user |
| `ServicePrincipalSignInLogs`   | App-to-app, service accounts, agents                    |
| `ManagedIdentitySignInLogs`    | Azure managed identities                                |
| `AuditLogs`                    | Role assignments, app registrations, consent grants     |

For an AI agent or third-party app integration, the agent's token flows will appear in `ServicePrincipalSignInLogs` and `NonInteractiveUserSignInLogs` — not in standard sign-in logs.

---

## What It Costs

Requires a **Log Analytics Workspace** in Azure (needs an active Azure subscription — PAYG is fine).

- Ingestion: ~$3–5 AUD/GB
- For Entra sign-in logs on a small tenant: **a few dollars/month at most**
- First 5GB/month per workspace is free

> **Decision needed:** requires boss sign-off as it incurs Azure costs, even if small.

---

## Setup Guide

### Step 1 — Create a Log Analytics Workspace

**Azure Portal → Log Analytics workspaces → Create**

- Resource group: use existing or create `rg-security`
- Region: match client's tenant region
- Name: e.g. `law-m365-logs`

### Step 2 — Enable Diagnostic Settings in Entra

**entra.microsoft.com → Monitoring & health → Diagnostic settings → Add diagnostic setting**

Tick:
- `AuditLogs`
- `SignInLogs`
- `NonInteractiveUserSignInLogs`
- `ServicePrincipalSignInLogs`
- `ManagedIdentitySignInLogs`
- `RiskyUsers` / `UserRiskEvents` (if Business Premium or P2)

Destination: **Send to Log Analytics workspace** → select workspace from Step 1.

Save. Logs start flowing within ~15 minutes.

### Step 3 — Verify

In Log Analytics → Logs:

```kql
SigninLogs
| take 10
```

```kql
AADNonInteractiveUserSignInLogs
| take 10
```

If rows return, it's working.

---

## Useful Queries

**All service principal sign-ins (last 7 days):**
```kql
AADServicePrincipalSignInLogs
| where TimeGenerated > ago(7d)
| project TimeGenerated, ServicePrincipalName, AppId, IPAddress, ResourceDisplayName, ResultType, ResultDescription
| order by TimeGenerated desc
```

**Guest/external user sign-ins:**
```kql
AADNonInteractiveUserSignInLogs
| where UserPrincipalName contains "theirdomain.com"
| project TimeGenerated, UserPrincipalName, AppDisplayName, IPAddress, ResourceDisplayName, ConditionalAccessStatus
| order by TimeGenerated desc
```

---

## Also Worth Checking (No Cost)

These don't need Log Analytics and are good practice regardless:

- **App consent granted to** — Entra → Enterprise Applications → [app] → Permissions
- **Admin consent workflow** — Entra → Identity → User settings → Admin consent requests → On
- **Conditional Access coverage for guests** — check if any CA policy applies to external/guest accounts
- **Defender for Cloud Apps OAuth monitoring** — if on Business Premium, security.microsoft.com → Cloud Apps → OAuth apps (policies off by default)

---

## Notes

- Requires Security Administrator or Global Administrator on the client tenant
- The Log Analytics workspace lives in the client's Azure subscription
- UAL must also be enabled separately — see [[M365 Unified Audit Log]]
- Not retroactive — only logs from when it's enabled onward
