---
category: research
tags:
  - obsidian
  - tasks-plugin
  - regex
date: 2026-04-01
---
# Obsidian Tasks — Regex Grouping by Custom Field

## The Problem

The Tasks plugin doesn't support arbitrary custom properties on individual tasks. Adding tags per task pollutes tag searches and the tag pane. The workaround is embedding a custom identifier in the task description and using `group by function` with a regex to extract it.

---

## The Solution — `||` Delimiter

Append `|| Client Name` to the end of any task you want grouped. Tasks without it fall into an "unsorted" block.

**Task format:**
```
- [ ] Rick to build a simple 'Clients' SharePoint Team Site || My Expert
- [ ] Follow up on printer issue || Bayside
- [ ] General unassigned task
```

---

## Query Setup

Two separate query blocks on the tasks page:

**Block 1 — Unsorted (no client):**
````
```tasks
not done
filter by function !task.description.includes("||")
```
````

**Block 2 — Grouped by client:**
````
```tasks
not done
filter by function task.description.includes("||")
group by function task.description.match(/\|\|(.+)/)?.[1]?.trim()
```
````

---

## How the Regex Works

`/\|\|(.+)/`

- `\|` — escaped pipe character (`|` has special meaning in regex so needs escaping)
- `\|\|` — matches literal `||`
- `(.+)` — capture group: everything after `||`
- `?.[1]` — pulls the first capture group from the match result
- `?.trim()` — strips any leading/trailing whitespace

---

## Why `||` Over `|`

Single `|` is used in Markdown tables and could cause parsing issues in some contexts. Double `||` is effectively never used naturally in a task name, making it a safe unambiguous delimiter.

---

## Useful Links

- [Obsidian Tasks — Custom Grouping](https://publish.obsidian.md/tasks/Scripting/Custom+Grouping)
- [Obsidian Tasks — Regular Expressions](https://publish.obsidian.md/tasks/Queries/Regular+Expressions)
- [Obsidian Tasks — Grouping](https://publish.obsidian.md/tasks/Queries/Grouping)
- [MDN — Regular Expressions Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions)
- [Regex101 — Online Regex Tester](https://regex101.com) — use JavaScript flavour to match how Tasks evaluates expressions
