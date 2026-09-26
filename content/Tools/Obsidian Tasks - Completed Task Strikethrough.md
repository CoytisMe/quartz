---
category: research
tags:
  - obsidian
  - tasksplugin
  - css
date: 2026-09-16
publish: false
---
## The Actual Mechanism

Completed checklist items (`- [x] ...`) get struck through and dimmed by **Obsidian's own core CSS, on by default**. Confirmed via Obsidian's own DevTools (Ctrl+Shift+I → inspect element → Styles panel shows the winning rule and its source file/line directly — the fastest way to answer "why does this render like X", see [[feedback_css_debugging_devtools_first]] for the general lesson):

```css
.markdown-source-view.mod-cm6 .HyperMD-task-line[data-task="x"],
.markdown-source-view.mod-cm6 .HyperMD-task-line[data-task="X"] {
  text-decoration: var(--checklist-done-decoration);
  color: var(--checklist-done-color);
}
```
(source: `app.css:14146`)

Reading view picks up the equivalent via `.task-list-item.is-checked`. This is genuinely not discoverable by grepping the shipped JS/CSS bundles for literal strings like `line-through` — the values are indirected through `var(--checklist-done-decoration)` etc., so the rendered behaviour's name never appears next to the selector.

---

## The Working Fix

`.obsidian/snippets/GoogleCompletedTasks.css` — found via a plain web search rather than reverse-engineering the app, and it just works, no `!important` needed:

```css
/* Affects Reading Mode */
.markdown-preview-view ul > li.task-list-item.is-checked,
.markdown-preview-view ol > li.task-list-item.is-checked {
  text-decoration: none;
  color: var(--text-normal);
}

/* Affects Editing/Live Preview Mode */
.markdown-source-view.mod-cm6 .HyperMD-task-line[data-task]:not([data-task=" "]) {
  text-decoration: none;
  color: var(--text-muted);
}
```

Targets the actual rendering selectors directly rather than overriding the `--checklist-done-*` CSS variables — beats Obsidian's default on specificity/source order alone. Live Preview intentionally uses `var(--text-muted)` (slightly dimmed) rather than fully normal text, tuned by Rick to taste — Reading Mode gets fully normal colour instead, but that's moot since Rick basically never uses Reading Mode.

**Deliberately doesn't touch Tasks-plugin query block (` ```tasks ` ) results.** Rick's query blocks are filtered to not-done tasks anyway, so a completed task never actually shows up there — nothing to style. The fix only needed to cover native checklists sitting directly in notes, for readability when looking back at a note later.

Enabled via `.obsidian/appearance.json`'s `enabledCssSnippets`. Syncs across devices via OneDrive like any other vault file.

---

## Dead Ends (kept so this isn't re-investigated the same way twice)

- **Adding a `text-decoration: line-through` rule to "fix" completed tasks not being struck through in a `tasks` query block.** Backwards — the default already IS struck through everywhere via the core rule above; nothing needed adding, only removing/overriding.
- **Overriding `--checklist-done-decoration`/`--checklist-done-color` directly at `:root`.** Correct variables, correct values, but lost the cascade fight against the core rule without `!important` — worth knowing that this override path needs forcing, whereas selector-based overrides (the working fix above) don't. Never got to confirm whether the `!important` version would have worked cleanly, since the Google snippet solved it first.
- **A `background: red !important` diagnostic confirmed a snippet loads and matches**, but doesn't reveal that a competing rule already sets the *same property to the same value* — `background` wasn't contested by anything, so it lit up fine regardless of whether the real fix (targeting `text-decoration`/`color`) was winning.

---

## Useful Links

- [Obsidian Tasks — Styling](https://publish.obsidian.md/tasks/Advanced/Styling)
- [Obsidian Help — CSS snippets](https://help.obsidian.md/Extending+Obsidian/CSS+snippets)
