---
category: tech
date: 2026-10-17
publish: true
tags:
  - kbdraft
  - publickb
  - exclaimer
  - outlook
  - email
  - signatures
---

## Exclaimer was a while thing
Client wanted the signatures to populate when drafting emails.

It doesn't

## Why: it's expected behaviour
Exclaimer has two client-side delivery methods, and they behave differently:

| Client | Method | Signature visible while drafting? |
|---|---|---|
| Outlook Classic | **Windows Agent** (Cloud Signature Update Agent) | Yes. It injects the signature client-side |
| New Outlook / OWA / mobile | **Outlook Add-in** | No. The signature is applied server-side on send |

The client app/add-in shows the signatures and lets users choose one, but **doesn't automatically put it at the bottom of a draft**. For New Outlook users that's an Exclaimer platform limitation, not a config issue. Set the expectation with the client rather than chasing it.

**Gotcha:** a user whose signature "works" while drafting may just have an old signature saved in Outlook, not a live Exclaimer one. Check before using them as the reference.

## Deploying the Windows Agent
Client downloads: https://support.exclaimer.com/hc/en-gb/p/outlook-client
Agent deployment doc: https://support.exclaimer.com/hc/en-gb/articles/4406429578269-Install-Exclaimer-Signature-Update-Agent-Windows

```cmd
curl -o C:\exclaimer.msi https://outlookclient.exclaimer.net/csua/Exclaimer.CloudSignatureUpdateAgent.Install.msi
msiexec /i C:\exclaimer.msi ALLUSERS=1
```
*(The argument order in the source note was `curl -o <url> <file>`. Corrected here to `-o <file> <url>`. Test before publishing.)*

## Rollout checklist
- SPF: add `include:spf.au.exclaimer.net` (AU region) to the domain's SPF record.
- Admin consent: Exclaimer registers **six enterprise apps** in the tenant. Grant admin consent to all of them.
- Afterwards, remove any setup-only app, e.g. the one named "Exclaimer Cloud AU Setup - Please remove after setup".
- Test in OWA, then in Classic Outlook with the agent.
