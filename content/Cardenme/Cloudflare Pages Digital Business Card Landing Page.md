---
category: research
tags:
  - cloudflare
  - web
  - nfc
  - digital-card
date: 2026-05-10
publish: true
---

## Intro
Here's the info on the swappable digital business card I make for me as a person, me as a tech and me as a Coytis.

## The Setup

Cloudflare Pages has a free tier that handles this perfectly. It supports "Direct Upload" — you don't need Git, a terminal, or a build process. You just zip up a folder containing an `index.html` file (and any CSS/images) and drag it into the Cloudflare dashboard. The site goes live on a `<yourprojectname>.pages.dev` subdomain immediately, and you then attach your custom domain from the same dashboard. Since your domains are already on Cloudflare, attaching them is a one-click process — Cloudflare sets the DNS records automatically.

## The Page Itself

A digital business card page is just a single `index.html` file — no backend, no framework needed. Mobile-first since anyone tapping an NFC card is on their phone. Keep it one screen, no scrolling. The whole thing is plain HTML and CSS in one file.

## Direct Upload Deploy

In the Cloudflare dashboard go to Workers & Pages → Create application → Pages → Direct Upload. Give the project a name, drag in your folder or zip, hit Deploy. Once live on the `.pages.dev` URL, go to Custom Domains, add your domain, Cloudflare wires up DNS automatically.

**Caveat:** If you start with Direct Upload you can't switch to Git-connected deployment later — you'd need a new project.

---

## Swappable Card Setup (Git-connected)

A better approach for cards that need multiple variants — connect to GitHub and use a swap script to change which variant is live.

### Repo Structure

```
DigitalCard/
  index.html          ← active page (overwritten by swap script)
  variants/
    personal.html     ← rickhugecock.work personal (dark, minimal)
    work.html         ← work.rickhugecock.work Fuckingyourmomfactory details (white, green)
    creator.html      ← cardenme.com creator card (dark, pink)
  assets/
    worklogo.png
  .gitignore
  README.md
```

### Swapping Variants

```powershell
.\Other\swap-card
```

Lists available variants, prompts for a choice, copies to `index.html`, commits and pushes. Cloudflare redeploys in ~30 seconds.

### First Time Setup

```
cd "C:\Users\Rick\OneDrive - Coytis\Claude\DigitalCard"
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/CoytisMe/digital-card.git
git push -u origin main
```

Then connect to a new Cloudflare Pages project via GitHub (not Direct Upload).

---

## Cards Built

**personal** — `rickhugecock.work` — dark (#0d0d0d), minimal. Name: Rick Hugecock, tagline: EXISTS. Phone + email.

**work** — `work.rickhugecock.work` — white card, green accent (#2d7a3a). Fuckingyourmomfactory logo, Support Technician. Phone, email, SMS.

**creator** — `cardenme.com` — dark with pink accent (#c4296e). Name: Coytis, tagline: Attempted Creator. Email, TikTok, Instagram, YouTube, howto.coytis.me.

---

## The cardenme.com Angle

`cardenme.com` is a neutral domain — no name, no brand — suitable for offering this as a service to clients or using for contexts where you can't use your own name. Cards can be served from:

- `cardenme.com` — root, anonymous creator card
- `name.cardenme.com` — per-person subdomains for clients
- Each subdomain = a separate Cloudflare Pages project with a custom domain attached

Domains are ~$10/year. Each Pages project is free. Viable as a small client offering.

---

**Sources**

- [Cloudflare Pages — Direct Upload](https://developers.cloudflare.com/pages/get-started/direct-upload/)
- [Cloudflare Pages — Static HTML deploy](https://developers.cloudflare.com/pages/framework-guides/deploy-anything/)
- [Cloudflare Pages — Custom Domains](https://developers.cloudflare.com/pages/configuration/custom-domains/)
