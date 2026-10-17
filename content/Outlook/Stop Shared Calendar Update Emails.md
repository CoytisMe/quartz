---
category: tech
date: 2026-10-17
publish: true
tags:
  - kbdraft
  - publickb
  - outlook
  - calendar
  - m365
source:
---
# Stop Shared Calendar Update Emails

## Symptom
After shared calendars are added, a user starts getting calendar update emails from `no-reply@microsoft.com`.

## Fix
It's a notification setting, and it's set per user in OWA:

**Outlook on the web > Settings > Calendar > Shared calendars** > turn off the update notifications.

Confirmed working.

Related: [[Update Visibility on Room Calendars]]

## Tip
Anyone else who has the same shared calendars added will get the same emails. Switch the setting off for them too before they log their own tickets.
