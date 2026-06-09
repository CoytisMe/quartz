---
category: reference
tags:
  - digital-card
  - html
  - web
  - cloudflare
date: 2026-05-10
publish: true
---
# Digital Card — Customisation Possibilities

## Visual Editors

**VS Code** — free, lightweight, has live preview with the Live Server extension. Type HTML, see it update in the browser instantly. Best option for this use case, not overkill at all. Install: code.visualstudio.com

**CodePen** (codepen.io) — browser-based, paste your HTML/CSS in and see it render live. No install, good for quick experiments before committing to the file. Free tier is fine.

**Dreamweaver** — still exists but overkill and paid. VS Code does the same job for free.

---

## What Can Be Added

### Contacts & Actions
- **Save to contacts button** — downloads a `.vcf` file, one tap and the person has your full contact saved. Very useful on the work card
- **Click to copy** — tap phone number or email and it copies to clipboard with a confirmation flash
- **Open in Maps** — address link that opens Apple Maps or Google Maps depending on the device

### Content
- **Profile photo** — circular avatar, very common on card pages
- **Custom fonts** — Google Fonts, load any typeface you want
- **Tagline or bio** — short line of text under the name
- **QR code** — generates on the page, useful for sharing the URL itself

### Interactions
- **Animations** — fade in on load, hover effects, button press feedback
- **Dark/light mode toggle** — detects system preference automatically or adds a manual toggle
- **Confetti or effects** — for personal/fun cards, why not

### Integrations
- **Calendly / booking link** — embed a scheduling button that opens their calendar
- **Social follower counts** — pull live numbers from APIs (requires a bit of backend or a free service like shields.io)
- **Linktree-style layout** — multiple links in a clean stacked list, already basically what the cards are

### Advanced
- **Analytics** — Cloudflare Pages has built-in analytics, or add a tiny script for page view counts
- **Password protection** — Cloudflare Access can gate the page behind a login if needed for private cards
- **A/B variants** — already have this with the swap script, could go further with Cloudflare rules

---

## Quickest Wins Worth Adding

1. **Save to contacts (.vcf)** — especially on the work card
2. **Click to copy** on phone number
3. **Profile photo** on the personal card
4. **Fade in animation** — one line of CSS, makes it feel polished

---

## Editing Workflow

1. Open the variant file in VS Code (`variants/work.html` etc)
2. Install the **Live Server** extension in VS Code
3. Right-click the file → Open with Live Server → edits show in the browser instantly
4. When happy, commit and push — `.\swap-card` or push manually
5. Cloudflare redeploys in ~30 seconds


