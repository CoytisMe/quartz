---
category: work-note
date: 2026-08-14
publish: true
tags:
  - outlook
  - leap
  - adobe
  - acrobat
  - troubleshooting
---

# LEAP Outlook — Email Body Not Showing (Only Attachments)

## Symptom

Emails opened from LEAP in Outlook don't display the email body — only the attachment list is visible.

Two details that help confirm it's this problem and not something else:

- It affects **received emails only**. Sent items with the same attachments open fine.
- It shows up on messages **with PDF attachments**.

That received-only pattern rules out the usual suspects — PDF preview handler, Protected View / Mark-of-the-Web on LEAP's temp path — because those would break sent items too.

## Cause

The **Adobe Document Cloud for Microsoft Outlook** add-in, registered as:

```
AdobeAcroOutlook.SendASLink
```

(Earlier version of this note blamed *Acrobat PDFMaker Office COM Addin*. That was wrong — confirmed August 2026 that `SendASLink` is the one.)

The add-in walks the attachment table on incoming messages to decide whether to offer its "save to Document Cloud" prompt. Sent items have already been materialised by the composer, so it never touches them the same way — hence received-only.

Nothing in LEAP depends on this add-in. Safe to kill permanently, provided the firm isn't actually using Adobe Document Cloud (most LEAP shops aren't — the matter store *is* the document store). Confirm before removing.

## Quick fix

Disable or uninstall the add-in in Outlook. Bodies come good immediately.

**This will not hold.** Adobe's ARM updater re-registers it on every Acrobat update and often just on Acrobat launch. Unchecking it in the COM Add-ins dialog only sets `LoadBehavior`, and the updater overwrites that. Setting `LoadBehavior` to `0` by hand doesn't survive either.

## Making it stick

Office policy overrides `LoadBehavior` entirely, so the updater can rewrite the key all it likes and Outlook ignores it.

Confirm the ProgID on the machine first — don't assume:

```
reg query "HKCU\Software\Microsoft\Office\Outlook\Addins" /s /f "Adobe"
```

Then apply the block:

```
reg add "HKCU\Software\Policies\Microsoft\Office\16.0\Outlook\Resiliency\AddinList" /v "AdobeAcroOutlook.SendASLink" /t REG_SZ /d "0" /f
```

Data value: `0` = always disabled, `1` = always enabled and the user can't turn it off.

Notes:

- `16.0` covers Office 2016 through M365.
- This is **user scope** — target it at users in GPO/Intune, not machines.
- The policy tree usually won't exist yet. `reg add` creates it. Regedit doesn't live-refresh, so hit F5 before assuming it failed.
- Restart Outlook fully (check Task Manager for a lingering `OUTLOOK.EXE`) — policy is only read at startup.

**Verification:** the entry in File → Options → Add-ins → COM Add-ins should be **greyed out and untickable**, not merely unticked. Unticked means the policy didn't apply.

## If policy doesn't hold

Deny `SetValue` on `HKCU\Software\Microsoft\Office\Outlook\Addins\AdobeAcroOutlook.SendASLink` so the updater physically can't flip `LoadBehavior` back to `3`. Blunt, but effective and it survives Adobe updates.

## Fleet-wide option

If this spreads beyond one or two users, strip the Outlook integration out of the Acrobat deployment itself — the Adobe Customization Wizard has a toggle for it, so the add-in never lands and there's nothing to re-enable. More work up front, but the policy approach is a per-user patch on every affected machine.

Not worth it for isolated cases. Policy fix and move on.

## Related

- [[LEAP Adobe Add-in Script]] — reinstalling LEAP's own Acrobat integration after Adobe breaks it
- [[Acrobat 26.001 PDF Not Opening]]

## History

- **2026-07-01** — first documented. Fix recorded as "uninstall Adobe add-ins".
- **2026-08-14** — recurrence on a single user. Identified the actual ProgID, confirmed received-only pattern, established that COM-dialog disable and `LoadBehavior=0` both get reverted by Adobe's updater. Policy `AddinList` block applied.
