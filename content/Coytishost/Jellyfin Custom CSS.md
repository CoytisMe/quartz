---
category: reference
date: 2026-06-06
publish: true
tags:
  - jellyfin
  - css
  - media-server
---

## I stole the colour scheme off my Coytis Page
The one for my digital business card, the one I never use, the one with the colour scheme that was copied from my BlueBerryPie Powershell theme, the one that was stolen from some dude on the internet.

Paste into **Dashboard → General → Custom CSS → Save**.

Sourced from the colour scheme of `DigitalCard/variants/creator.html`.

## Colour Palette

| Token | Hex | Used for |
|---|---|---|
| `--bg` | `#1C0C28` | Page background |
| `--surface` | `#2A1238` | Cards, header, sidebar |
| `--border` | `#3D1852` | Card borders, dividers |
| `--accent` | `#9D54A7` | Buttons, active states |
| `--accent-soft` | `#BC94B7` | Links, secondary text |
| `--accent-glow` | `#F27EF2` | Hover glow |
| `--text` | `#F0E8D6` | Primary text |
| `--link-hover-bg` | `#351048` | Hover backgrounds |

## CSS

```css
/* ── Coytis Creator Theme ── */
:root {
    --theme-dark-mode-bg:           #1C0C28;
    --theme-toolbar-bg:             #2A1238;
    --theme-button-bg:              #2A1238;
    --theme-button-hover-bg:        #351048;
    --theme-accent:                 #9D54A7;
    --theme-accent-light:           #BC94B7;
    --theme-text:                   #F0E8D6;
    --theme-text-muted:             #BC94B7;
}

/* Background */
.backgroundContainer, .background-image-container, body {
    background-color: #1C0C28 !important;
}

/* Cards / surfaces */
.card, .itemDetailPage, .paperList, .listItem,
.emby-input, .emby-select, .emby-textarea {
    background-color: #2A1238 !important;
    border: 1px solid #3D1852 !important;
    border-radius: 12px;
}

/* Top navigation / header */
.skinHeader, .headerRight {
    background-color: #2A1238 !important;
    border-bottom: 1px solid #3D1852 !important;
}

/* Accent — buttons, active states, highlights */
.raised, .raised.emby-button,
.button-accent, .button-accent-filled,
.fab, .headerButton.headerButtonRight.headerMenuButton {
    background-color: #9D54A7 !important;
    color: #F0E8D6 !important;
}

/* Links and interactive text */
a, .emby-button, .navMenuOption-selected {
    color: #BC94B7 !important;
}

a:hover, .emby-button:hover {
    color: #F27EF2 !important;
}

/* Progress / scrubber bars */
.mdl-slider::-webkit-slider-thumb,
.progressring circle,
.itemProgressBar div {
    background-color: #9D54A7 !important;
    border-color: #9D54A7 !important;
}

/* Sidebar / drawer */
.navMenuOption:hover, .navMenuOption:focus {
    background-color: #351048 !important;
}

/* Scrollbar */
::-webkit-scrollbar-thumb {
    background-color: #3D1852 !important;
}
::-webkit-scrollbar-thumb:hover {
    background-color: #9D54A7 !important;
}

/* Text */
.emby-input, .emby-select, .emby-textarea,
.cardText, .itemName, h1, h2, h3 {
    color: #F0E8D6 !important;
}

.secondary, .cardText-secondary, .textMuted {
    color: #BC94B7 !important;
}
```
