---
category: work-note
date: 2026-06-10
publish: true
tags:
  - adobe
  - acrobat
source: "[[10-06-26]]"
---
# Adobe Email Button Issue

**Found this is in my daily note, not actually sure it works**

Swapped a client's Adobe from new to old style UI because she wanted an email button. Got the button but it was auto-generating emails with "Download Adobe Reader here" text at the bottom.

The button should have a popout menu to choose send as link or attachment — that wasn't appearing, just auto-generated the email directly.

Didn't have time to fix it; client was happy to manually delete the footer text.

## Claude Followup

Worth finding the setting that controls the auto-generated email footer in Adobe Acrobat (old UI). Likely in Preferences > Email or a send/share setting. Could also be an Acrobat version issue — newer builds may have removed the send-as-link popout entirely. Check Adobe forums for "email button remove download reader footer acrobat". If it can't be suppressed, the new UI might actually be the better option for this client long-term.
