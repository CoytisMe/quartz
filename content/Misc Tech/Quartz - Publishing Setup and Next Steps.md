---
category: research
tags:
  - research
  - quartz
  - publishing
  - obsidian
date: 2026-04-15
publish: true
---
You do not want the know the fuckery that would've befallen had I not sorted out the 'default off' publishing shit, goddamn.

Notes from the initial setup session. words.coytis.me is live, pointing at Cloudflare Pages from the CoytisMe/quartz GitHub repo. Content is currently the Work/Microsoft folder with a manually copied sync workflow. Several things to improve.

---

## 1. Controlling What Gets Published (ExplicitPublish Plugin)

By default Quartz publishes everything in the `content` folder. To make `publish: false` (and `publish: true`) actually do something, enable the `ExplicitPublish` plugin.

In `quartz.config.ts`, find the `plugins` section and add it to the `filters` array:

```ts
filters: [
  Plugin.ExplicitPublish(),
],
```

With this enabled:
- Notes with `publish: true` → published
- Notes with `publish: false` → excluded
- Notes with no `publish` key → **excluded by default** (opt-in model)

This is the safest approach — nothing goes live unless you've explicitly said so. All templates and the Claude Briefing already have `publish: false` added.

---

## 2. Syncing the Vault to Quartz (OneDrive → GitHub)

The symlink approach didn't work because Git pushed the pointer rather than the files. Current workaround is manually copying the content folder. Better options:

### Option A — PowerShell sync script (recommended)
A script that robocopy's (or xcopy's) the relevant vault folders into the Quartz content folder, then runs the git add/commit/push pipeline. One script, one run.

```powershell
# Example structure
$source = "C:\Users\Rick\OneDrive - Coytis\Obsidian\Rick\Work\Microsoft"
$dest = "C:\Users\Rick\quartz\content"

robocopy $source $dest /MIR /XF "*.canvas"

Set-Location "C:\Users\Rick\quartz"
git add .
git commit -m "content sync $(Get-Date -Format 'yyyy-MM-dd HH:mm')"
git push
```

`/MIR` mirrors the source — deletions in vault also remove from content. Remove `/MIR` if you want additive-only sync.

### Option B — GitHub Actions
Set up an Action that pulls from OneDrive on a schedule or trigger. More complex, probably overkill.

### Option C — Quartz private content folder with .gitignore tricks
Keep content outside the repo and use a pre-build copy step. Also complex.

**Recommended: Option A.** Get the script working, save it to `C:\Scripts\`, wire it to a shortcut or StreamDeck button.

---

## 3. One-Button Publish Script

Full pipeline script — copy content, commit, push:

```powershell
# publish-quartz.ps1
param(
    [string]$CommitMessage = "content sync $(Get-Date -Format 'yyyy-MM-dd HH:mm')"
)

$source = "C:\Users\Rick\OneDrive - Coytis\Obsidian\Rick\Work\Microsoft"
$dest = "C:\Users\Rick\quartz\content"
$repo = "C:\Users\Rick\quartz"

Write-Host "Syncing content..." -ForegroundColor Cyan
robocopy $source $dest /MIR /XF "*.canvas" | Out-Null

Write-Host "Committing and pushing..." -ForegroundColor Cyan
Set-Location $repo
git add .
git commit -m $CommitMessage
git push

Write-Host "Done. Cloudflare will deploy in ~1 min." -ForegroundColor Green
```

Save to `C:\Scripts\publish-quartz.ps1`. Run with:
```powershell
C:\Scripts\publish-quartz.ps1
```

Or add a custom commit message:
```powershell
C:\Scripts\publish-quartz.ps1 -CommitMessage "added exchange notes"
```

For StreamDeck: point a button at a `.bat` file that calls the script.

---

## 4. Styling — Getting It Cleaner

Quartz is styled via `quartz\styles\custom.scss`. It already looks decent but a few tweaks get it closer to Obsidian's feel.

### Font
Obsidian uses Inter by default. Add to `custom.scss`:

```scss
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&display=swap');

:root {
  --bodyFont: "Inter", sans-serif;
  --headerFont: "Inter", sans-serif;
}
```

### Colour scheme
Quartz uses CSS variables. Override in `custom.scss` to match Obsidian's dark theme:

```scss
:root[saved-theme="dark"] {
  --background: #1e1e2e;
  --darkgray: #cdd6f4;
  --gray: #a6adc8;
  --lightgray: #313244;
  --highlight: #45475a;
  --tertiary: #89b4fa;
  --secondary: #cba6f7;
}
```

These are Catppuccin Mocha values — close to what you've already set up in Windows Terminal.

### Other useful tweaks
- `quartz.layout.ts` — controls sidebar, table of contents, backlinks placement
- Can hide the graph view if you don't want it public
- Can disable the explorer sidebar for a cleaner single-column layout

---

## Sources

- [Quartz documentation](https://quartz.jzhao.xyz)
- [Quartz hosting guide](https://quartz.jzhao.xyz/hosting)
- [ExplicitPublish plugin docs](https://quartz.jzhao.xyz/plugins/ExplicitPublish)
- [Quartz styling guide](https://quartz.jzhao.xyz/configuration)
- [Cloudflare Pages docs](https://developers.cloudflare.com/pages/)
