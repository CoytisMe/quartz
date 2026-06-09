---
category: research
tags:
  - research
  - quartz
  - obsidian
  - web-hosting
  - cloudflare
  - github
  - coytis
date: 2026-04-14
publish: true
---

## Ever wondered how this site was made?
Fuck if I know, here's what Claude said

(it was originally words.coytis.me)

Static site generator built specifically for publishing Obsidian vaults to the web. Free, open source, self-hostable. Handles wikilinks, backlinks, graph view, callouts, and Obsidian-flavoured Markdown natively — no translation layer needed.

The main alternative to Obsidian Publish ($10–20/month). Quartz is free but requires a bit of setup.

**Official docs:** https://quartz.jzhao.xyz

---

## How It Works

Quartz is a static site generator — it takes your Markdown notes and builds them into a folder of plain HTML/CSS/JS files. Those files get deployed to a host (Cloudflare Pages, GitHub Pages, etc.) and served as a website. There's no server running, no database — just files.

The build pipeline is:

1. Your Obsidian vault (or a subset of it) is the content source
2. Quartz reads the Markdown and outputs a static site
3. That output is pushed to a hosting platform
4. The hosting platform serves it at a URL
5. You point your domain at that URL via DNS

You run the build manually (one command), or set it up to build automatically every time you push to GitHub.

---

## Choosing What to Publish

This is the key question, and the answer is: **you have full control, and it's not hard.**

### Option A – Publish everything (default)
By default Quartz publishes all Markdown files in the content folder. Simplest approach if your vault is already curated or you're not worried about privacy.

### Option B – Opt-in per note (`publish: true`)
Add a filter in `quartz.config.ts` using the built-in `ExplicitPublish` plugin. Only notes with `publish: true` in their frontmatter get published. Everything else is ignored.

```yaml
---
publish: true
---
```

This is the recommended approach for a mixed private/public vault. You can tag notes for publish in Obsidian and the filter handles the rest. There's also a `RemoveDrafts` filter for keeping work-in-progress notes out even if they're otherwise marked for publish.

### Option C – Folder-based filtering
Use `ignorePatterns` in `quartz.config.ts` to exclude entire folders by glob pattern:

```ts
ignorePatterns: [
  "Private/",
  "Dailys/",
  "Work Clients/",
  "**/*.pdf"
]
```

This is good for keeping whole sections of the vault out of the build — e.g. client notes, daily notes, personal stuff — while publishing the rest.

### Mixing approaches
You can combine both: exclude folders by default, and use `publish: true` for individual notes you want to surface from folders that are otherwise excluded. Most people end up doing this.

**Important caveat:** Filters only apply to Markdown files. Non-Markdown files (images, PDFs, attachments) are copied to the output folder by default even if no note links to them. If your vault contains private attachments, explicitly exclude file types or attachment folders in `ignorePatterns`.

---

## Adding or Removing Content

Adding or removing notes from the published site is just:

1. Add `publish: true` to a note's frontmatter (or remove it)
2. Run `npx quartz sync` — this rebuilds and pushes the updated site

That's it. There's no separate CMS, no upload step, no dashboard. The vault is the source of truth.

If you're using folder-based filtering, adding or excluding whole folders is one line change in the config, then a sync.

---

## Syncing and Publishing Workflow

**Quartz does not auto-sync with Obsidian.** It's not a live mirror. You publish when you choose to by running a command. Two modes:

### Manual publish (simplest)
Run from the Quartz folder whenever you want to push updates:

```bash
npx quartz sync
```

This builds the site and pushes the output to GitHub. If Cloudflare Pages is connected to that GitHub repo, it picks up the push and deploys automatically — usually within a minute.

So the full flow is:
- Write/edit in Obsidian as normal
- When ready: `npx quartz sync`
- Site updates

### Automated publish (optional)
Can be set up with GitHub Actions to trigger a build whenever you push your vault to GitHub. Requires your vault to be in a GitHub repo (private is fine). More setup, but means the site stays in sync with your vault automatically.

Not necessary for a personal site — manual sync is low friction.

---

## Hosting Options

### Cloudflare Pages (recommended)
Free tier, fast, global CDN, handles custom domains and SSL automatically. If coytis.me is already on Cloudflare (which it likely is if you're using Cloudflare for DNS), this is the easiest path — everything stays in one place.

**Build settings for Cloudflare Pages:**
- Framework preset: None
- Build command: `npx quartz build`
- Build output directory: `public`

Connect it to your GitHub repo and every push triggers a new deploy.

### GitHub Pages (alternative)
Also free, slightly more fiddly with trailing slashes and URL formatting. Fine if you prefer to stay in GitHub ecosystem. Quartz's docs recommend Cloudflare Pages over GitHub Pages specifically because of the trailing slash issue.

---

## Pointing coytis.me at a Quartz Site

### Full domain (coytis.me → vault)

If the whole site is the Quartz vault:

1. Deploy Quartz to Cloudflare Pages → get a `*.pages.dev` URL
2. In Cloudflare Pages dashboard: Workers & Pages → your project → Custom Domains → Set up custom domain → enter `coytis.me`
3. Cloudflare adds the DNS records and provisions SSL automatically — nothing to do manually

That's the whole DNS step. If coytis.me is already managed by Cloudflare DNS, it literally just works.

### Subdirectory (coytis.me/words → vault)

This is the more interesting option for the "business card + vault" idea. A subdirectory path like `coytis.me/words` pointing to a Quartz site is **not straightforward** with static hosting.

The problem: Cloudflare Pages deploys to either a full domain or a subdomain (`words.coytis.me`), not a subdirectory path of another site. A subdirectory like `/words` requires either:

- A single Cloudflare Pages project that contains both the landing page and the Quartz output (awkward to maintain)
- A Cloudflare Worker that rewrites requests to `/words/*` to the Quartz Pages deployment (more elegant, small amount of Worker code)
- A reverse proxy setup (overkill for this scale)

The **subdomain approach** (`words.coytis.me`) is much simpler and essentially free — just point a subdomain CNAME at the Quartz Pages deployment. Functionally equivalent for most purposes.

**Recommended path for the coytis.me idea:** Start with `words.coytis.me` for the vault, build the landing page at `coytis.me` separately (static HTML, another Pages project, or even a simple Cloudflare Worker). If the subdirectory path matters later, wire it up with a Worker redirect.

---

## The coytis.me "Business Card + Vault" Architecture

The idea: a simple landing page at `coytis.me` with socials and basic info, vault accessible as a secondary destination.

Recommended setup:

| Component | What | How |
|---|---|---|
| `coytis.me` | Landing page | Static HTML deployed to Cloudflare Pages |
| `words.coytis.me` | Quartz vault | Separate Cloudflare Pages project, custom subdomain |
| DNS | Cloudflare | CNAME for subdomain, both get SSL automatically |

The landing page doesn't need to be complex — a single `index.html` with your name, a line about what you do, links to Twitch/TikTok/YouTube, and a link to `words.coytis.me`. Deployable in an afternoon.

The vault sits at the subdomain, independently maintained. You can link to specific notes from the landing page when you want to share something.

---

## What Quartz Supports Out of the Box

- Wikilinks (`[[...]]`) — resolved natively
- Backlinks — shown on every page
- Graph view — interactive note graph
- Full-text search
- Obsidian callouts (`> [!note]`, `> [!warning]`, etc.)
- Frontmatter (tags, date, title)
- Mermaid diagrams
- LaTeX
- Syntax highlighting for code blocks
- Popover previews on hover
- Dark/light mode

What it does not do:
- Real-time sync (manual publish step required)
- Per-note password protection
- Comments (needs third-party integration)

---

## Setup Prerequisites

- Node.js v22+ and npm v10.9.2+ installed (Node is already installed on your machine)
- A GitHub account and a repo for the Quartz project
- Git installed and basic familiarity with `git push`
- Cloudflare account (free) with coytis.me already managed there

---

## Open Questions

- Is coytis.me's DNS currently managed through Cloudflare, or through the domain registrar directly? This affects how easy the custom domain step is.
- Which subset of the vault would be worth publishing? Research notes and Work KB are the obvious candidates; daily notes and client folders definitely out.
- Would the vault publish under a real name/brand or stay anonymous? Affects how much care goes into the index/landing page.

---

## Sources

- [Quartz 4 Official Docs](https://quartz.jzhao.xyz)
- [Quartz Hosting Guide](https://quartz.jzhao.xyz/hosting)
- [Quartz Obsidian Compatibility](https://quartz.jzhao.xyz/features/Obsidian-compatibility)
- [Cloudflare Pages Custom Domains](https://developers.cloudflare.com/pages/configuration/custom-domains/)
- [Oliver Falvai – Private + Public Vault Setup with Quartz](https://oliverfalvai.com/evergreen/my-quartz-+-obsidian-note-publishing-setup)
- [Rakshan Shetty – Multi-Site Publishing with Quartz](https://rakshanshetty.in/blog/quartz-obsidian-multi-site-publishing)
