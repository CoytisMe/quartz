---
category: tech
date: 2026-10-17
publish: true
tags:
  - kbdraft
  - publickb
  - avanan
  - email
  - security
  - phishing
---
## Avanan Bullshit


## Held attachments expire (default 14 days)

- Avanan holds password-protected attachments it can't scan and emails the user a link to unlock them.
- **Default retention on held items is 14 days.** After that the control panel says the attachment is inaccessible due to retention, and it **can't be recovered**.
- Users often don't click the unlock link until weeks later, by which point it's gone.
- Retention can be raised per tenant (raised to 6 months on one tenant after a loss). Depending on how the licence was bought, the change may have to go through the distributor.

**Do:** raise retention on every tenant up front rather than waiting for the next lost attachment, and tell users that held-for-unlock items expire, so they don't sit on the link.

## Password-protected attachments that won't open

- Release the original to the user to test. If the original opens, the Avanan unlock link should work too.
- In the case seen, both worked once tested. The user had the wrong password for the file.
- So check the password before blaming Avanan.

## "Everything's clean" isn't the same as safe, I got scammed and told everyone else shit was cash

Case: an email quarantined on wording only (an over-excited quote request, CC'd to everyone).

- SPF, DKIM and DMARC all passed
- The attachment scan was clean
- There was previous legitimate correspondence with the sender

It was released, and it turned out the scam link was **inside a Dropbox link inside the attachment**. Auth checks and the attachment scan only clear the first layer. The user caught it and it was re-quarantined.

**Do:** before releasing anything flagged on wording, open the attachment somewhere safe and follow any links it contains, including file-share links.

## Fin
I still haven't checked if they let you pull attachments but if they do an EOD VM or Linux machine is a good call 
