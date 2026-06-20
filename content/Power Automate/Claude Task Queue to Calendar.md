---
category: work-note
date: 2026-06-10
publish: true
tags:
  - power-automate
  - tasks
  - automation
  - claude
---
## This was my greatest or worst idea, not sure

I wanted Claude to be able to add tasks to my MS ToDo, I moved over from Google Tasks, can't remember why.

All the Google and MS MCPs and shit constantly needed to be reconnected, the 365 web connector in clade couldn't make tasks.

So it's a power automate flow that watched a .txt file on my OneDrive that Claude can edit, when it does so the PA flow reads it, extracts the tasks, and resets it.

Works like a charm

**Queue file:** `C:\Users\Rick\OneDrive - Coytis\Obsidian\Rick\Tasks\ToDoQueue.txt`

---

## How It Works

1. Claude appends tasks to `ToDoQueue.txt` between the comment markers
2. PA detects the file change on OneDrive (watches the Tasks folder)
3. PA checks the filename is `ToDoQueue.txt` — terminates if not
4. PA reads the file and extracts lines between `<!-- QUEUE START -->` and `<!-- QUEUE END -->`
5. PA checks if the queue is empty — terminates if so (prevents infinite loop from its own clear operation)
6. For each task line, PA creates a task in Microsoft To Do
7. PA rewrites the file with empty markers (clears the queue)
8. Flow fires again on the clear, finds empty queue, terminates cleanly

---

## Task Format

```
- Task title 📅 YYYY-MM-DD
- Task title (no date)
```

The 📅 date is optional. Tasks without a date land in To Do with no due date set.

---

## Flow Structure

### Trigger
- Connector: **OneDrive for Business**
- Action: **When a file is modified**
- Folder: `Obsidian/Rick/Tasks/` (watches the whole folder)

### Step 1 — Filename check
- **Condition:** `triggerOutputs()?['body/Name']` is equal to `ToDoQueue.txt`
- No → **Terminate** (Succeeded)
- Yes → continue

### Step 2 — Get file content
- Connector: **OneDrive for Business**
- Action: **Get file content** — browse to `ToDoQueue.txt` via the folder picker (do not type the path)

### Step 3 — Extract queue
- **Compose:** extract text between markers:
```
split(split(body('Get_file_content'),'<!-- QUEUE START -->')[1],'<!-- QUEUE END -->')[0]
```
- Split into lines:
```
split(trim(outputs('Compose')), '\n')
```

### Step 4 — Filter empty lines
- **Filter Array** — keep only lines where `startsWith(item(), '- ')`

### Step 5 — Empty queue check
- **Condition:** `empty(trim(outputs('Compose')))` is equal to `true`
- Yes → **Terminate** (Succeeded)
- No → continue

### Step 6 — Apply to each task line
Inside the loop:

**Condition:** `contains(items('Apply_to_each'), '📅')`

- **Yes branch — task with due date:**
  - Title: `trim(replace(trim(first(split(first(split(item(), '📅')), '||'))), '- ', ''))`
  - Due date: `formatDateTime(trim(first(split(last(split(item(), '📅')), '||'))), 'yyyy-MM-dd')`

- **No branch — task without due date:**
  - Title: same title expression as above
  - Due date: leave blank

Both branches use **Microsoft To Do (Business)** → **Add a to-do (V3)**

### Step 7 — Clear the queue
- Connector: **OneDrive for Business** → **Update file**
- Rewrite with empty markers (see queue file template below)

---

## Queue File Template

```
<!-- QUEUE START -->
<!-- QUEUE END -->
```

The file doesn't need frontmatter or headings — PA only cares about the markers.

---

## Tips

- **Paste the 📅 emoji directly** into expression fields — don't try to type it
- **Browse for the file** in Get file content — don't type the path or it'll return Invalid fileId
- The self-terminating empty check is what prevents the infinite loop after clearing
- The To Do connector must be signed in as `coytis@coytis.me`
- All connectors used are free on Business Premium

---

## Status
- [x] Queue file created
- [x] PA flow built
- [x] Tested end to end — working
- [ ] Claude briefed to use the queue format
