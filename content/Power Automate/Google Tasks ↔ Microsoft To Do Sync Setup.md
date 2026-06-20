---
title: Google Tasks ↔ Microsoft To Do Sync Setup
category: Work/Power Automate
tags:
  - power-automate
  - google-tasks
  - microsoft-todo
  - sync
created: 2026-05-24
publish: true
---

### Warning
Turned out there's no 'mark completed' for the Google Tasks trigger so you can't get proper 2 way sync, but it's sick for passing one to the other.

## Overview

Four flows for complete bidirectional sync. Loop prevention uses the tag `[S]` in the task body/notes — each flow checks for it and skips if present, preventing bounced triggers.

**Flow 1** (✅ DONE, needs modification) — Google Tasks → MS To Do (new tasks)  
**Flow 2** (to build) — MS To Do → Google Tasks (new tasks)  
**Flow 3** (to build) — MS To Do completed → Google Tasks completed  
**Flow 4** (to build) — Google Tasks completed → MS To Do completed  

All flows are at: https://make.powerautomate.com/environments/Default-4f2fc910-3444-4c0d-85f5-439cbb5011ea/flows

---

## Flow 1 — Modify existing: Google Tasks → MS To Do

Flow name: **Google Tasks → Microsoft To Do**  
Direct link: https://make.powerautomate.com/environments/Default-4f2fc910-3444-4c0d-85f5-439cbb5011ea/flows/361796af-aab9-4779-ac00-08799a7629ae

### Changes needed to existing flow

Open the flow, click Edit, switch to classic designer (toggle "New designer" off).

**Step 1 — Insert a Condition between the trigger and "Add a to-do (V3)"**

Hover over the arrow between the two steps → click the `+` → Add an action → Control → Condition.

Set the condition:
- Left side: click in the field → Add dynamic content → select **Notes** (from the Google Tasks trigger)
- Operator: **contains**
- Right side: type `[S]`

In the **Yes** branch (Notes contains [PA-SYNCED]):
- Add action → Control → **Terminate**
- Status: Succeeded (just stops the flow silently)

In the **No** branch (Notes does NOT contain [PA-SYNCED]):
- Move the existing "Add a to-do (V3)" step here by clicking `...` → Move down, or delete and recreate it inside the No branch.

**Step 2 — Update "Add a to-do (V3)" to include the sync tag**

In the "Add a to-do (V3)" action, fill in **Body Content**:
- Type: `[S]`

This marks the MS To Do task so Flow 2 doesn't bounce it back to Google.

Save.

---

## Flow 2 — New: MS To Do → Google Tasks (new tasks)

Flow name: **Microsoft To Do → Google Tasks**

### Build from scratch

Go to Create → new Automated cloud flow (classic designer).

**Trigger:** Search for `Microsoft To-Do` → select **"When a to-do is created (Business)"**
- Sign in with Microsoft account (coytis@coytis.me — should auto-connect)
- To-do List: **Tasks**

**Step 1 — Condition (loop prevention)**

Add action → Control → Condition:
- Left side: dynamic content → **Body Content** (from the To Do trigger)
- Operator: **contains**
- Right side: `[S]`

**Yes branch** (came from sync — ignore):
- Control → **Terminate** → Succeeded

**No branch** (genuinely new task — sync it):

Add action → search **Google Tasks** → **"Create a task"** (or similar)
- Task List: **My Tasks**
- Title: dynamic content → **Subject** (or Title from the To Do trigger)
- Notes: type `[S]`  ← loop prevention tag
- Due Date: dynamic content → **Due Date/Time** (if available)

Save as: **Microsoft To Do → Google Tasks**

---

## Flow 3 — New: MS To Do completed → Google Tasks completed

Flow name: **MS To Do Completed → Google Tasks Completed**

> **Note:** There is no "completed" trigger in the MS To Do connector — only **"When a to-do is updated (Business)"**. Use that trigger and add a condition to check the status field.

### Build from scratch

Create → new Automated cloud flow (classic designer).

**Trigger:** Search `Microsoft To-Do` → **"When a to-do is updated (Business)"**
- To-do List: **Tasks**

**Step 1 — Check if the task was actually completed**

Add action → Control → Condition:
- Left side: dynamic content → **Status** (from the trigger)
- Operator: **is equal to**
- Right side: type `completed`

**No branch** (task was updated but not completed — ignore):
- Control → **Terminate** → Succeeded

**Yes branch** (task is now completed — continue):

**Step 2 — List Google Tasks to find the match**

Add action → search **Google Tasks** → **"List tasks in a task list"**
- Task List: **My Tasks**

**Step 3 — Apply to each** (loop through Google tasks to find title match)

Add action → Control → **Apply to each**
- Select output from previous step: **value** (the list of tasks)

Inside the loop, add a Condition:
- Left side: dynamic content → **Title** (current Google task in the loop)
- Operator: **is equal to**
- Right side: dynamic content → **Subject** (from the MS To Do trigger — the completed task's title)

**Yes branch** (titles match — this is the one to complete):

Add action → Google Tasks → **"Update a task"** (or "Complete a task")
- Task List: **My Tasks**
- Task ID: dynamic content → **Id** (from the current Google task in the loop)
- Status: **completed**

**No branch:** leave empty.

Save as: **MS To Do Completed → Google Tasks Completed**

---

## Flow 4 — New: Google Tasks completed → MS To Do completed

Flow name: **Google Tasks Completed → MS To Do Completed**

### Build from scratch

Create → new Automated cloud flow (classic designer).

**Trigger:** Search **Google Tasks** → **"When a task is completed in a task list"**
- Task List: **My Tasks**

**Step 1 — Loop prevention check**

Add action → Control → Condition:
- Left side: dynamic content → **Notes** (from trigger)
- Operator: **contains**
- Right side: `[S]`

**Yes branch:** Control → Terminate → Succeeded

**No branch:**

**Step 2 — List MS To Do tasks to find the match**

Add action → **Microsoft To-Do (Business)** → **"List to-do's by folder (V2)"**
- To-do List: **Tasks**

**Step 3 — Apply to each**

Control → Apply to each → output from List to-do's step (the **value** array)

Inside loop → Condition:
- Left side: dynamic content → **Subject** (current MS To Do task title)
- Operator: **is equal to**
- Right side: dynamic content → **Title** (from Google Tasks trigger)

**Yes branch:**

Add action → **Microsoft To-Do (Business)** → **"Update to-do (V2)"**
- To-do List: **Tasks**
- To-do ID: dynamic content → **Id** (current MS To Do task in loop)
- Status: **completed**

**No branch:** leave empty.

Save as: **Google Tasks Completed → MS To Do Completed**

---

## Notes

- The `[S]` tag in body/notes is the only loop guard — don't remove it from any flow.
- Flow 2 depends on MS To Do having a "When a to-do is created" trigger — verify this exists when building. If it doesn't exist, Flow 2 can be done as a scheduled flow (every 15min, list all To Do tasks, check which ones aren't in Google Tasks yet).
- Flows 3 and 4 use "Apply to each" + title matching, which is slightly slow but reliable. The alternative is to store Google Task ID in the MS To Do body during creation (Flow 1), then use that ID directly instead of title-matching.
- After building all 4 flows, test by adding a task in Google Tasks and confirming it appears in MS To Do within ~1 min, and vice versa.
