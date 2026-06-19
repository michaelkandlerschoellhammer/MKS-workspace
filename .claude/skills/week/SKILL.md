---
name: week
description: Monday planning (set weekly goals + focus tasks) or Friday wrap-up (report progress, goals met, carry-forward). Also maintains Dashboard/Weekly.md.
argument-hint: "[plan|close|status] — defaults to mode detection"
---

# Week

You help the user plan their week on Monday and wrap it up on Friday. Mode is auto-detected from the day of the week and the state of the current week's note.

## Storage

- **Weekly notes**: `Weeks/YYYY/YYYY-WNN.md` (e.g., `Weeks/2026/2026-W25.md`)
- **Dashboard**: `Dashboard/Weekly.md` (generated; shows current week's goals + progress)
- **Template**: `templates/weekly-note.md`

Week numbers use ISO 8601 (Monday = start of week). Week number is zero-padded to 2 digits.

---

## Mode Detection

Fetch the current week note before any user interaction.

| Condition | Mode |
|-----------|------|
| Note doesn't exist OR goals section is empty | **Monday Planning** |
| Note exists + goals filled + Friday wrap-up section is empty | **Friday Wrap-up** |
| Note exists + goals filled + it's not Friday | **Status Check** |

If the user passes an explicit argument (`plan`, `close`, `status`), use that mode regardless.

---

## Mode: Monday Planning

### Phase 1: Fetch context (no interaction)

Do ALL in parallel:
- Check if `Weeks/YYYY/` folder exists; create it if not
- Create the weekly note from `templates/weekly-note.md` if it doesn't exist yet
  - Fill `week:` frontmatter with `YYYY-WNN`
  - Fill the `# Week WNN · YYYY (Mon DD MMM – Fri DD MMM)` heading with real dates
  - Fill today's daily note link in the `📝 Daily Notes` section
- Read `Dashboard/Tasks.md` to get in-progress and high-priority backlog tasks
- Read last week's note (if it exists) to find carry-forward items from `➡️ Carry-forward to next week`

### Phase 2: Propose weekly goals (1 round)

Present:

```
Week WNN planning — Mon DD MMM to Fri DD MMM

Carry-forward from last week:
- [item from previous Friday wrap-up carry-forward, or "none"]

Active tasks to consider:
🔥 In Progress:
- [[T-NNN - Title]] (P1, due Fri)
- [[T-NNN - Title]] (P2)

📋 High-priority backlog:
- [[T-NNN - Title]] (P2, due next week)
- [[T-NNN - Title]] (P3)

Suggested weekly goals (3–5 max):
1. [Outcome-oriented goal] → [[T-NNN - Title]]
2. [Outcome-oriented goal] → [[T-NNN - Title]]
3. [Outcome-oriented goal]

Which tasks are your focus this week? Confirm goals or tell me what to change.
```

**Goal writing rules:**
- Goals are outcomes, not activities ("Ship X", "Decide Y", "Unblock Z") — not "Work on X"
- Each goal optionally links to 1–2 tasks with `→ [[T-NNN - Title]]`
- Carry-forward items from last week become candidate goals
- Max 5 goals. Challenge if user wants more.

### Phase 3: Save + generate dashboard (no interaction)

After confirmation:
- Write goals into the weekly note's `## 🎯 Weekly Goals` section
- Fill the `## 📋 Focus Tasks this week` table with linked tasks
- Regenerate `Dashboard/Weekly.md` (see **Dashboard Regeneration** below)

### Phase 4: Summary

```
Week WNN planned — Mon DD MMM to Fri DD MMM

Goals:
1. [Goal]
2. [Goal]
3. [Goal]

Focus tasks: N linked
Dashboard updated: Dashboard/Weekly.md
Weekly note: Weeks/YYYY/YYYY-WNN.md
```

---

## Mode: Friday Wrap-up

### Phase 1: Fetch context (no interaction)

Do ALL in parallel:
- Read current week note (goals + focus tasks)
- Read `Dashboard/Tasks.md` to check which focus tasks are done vs. still active
- Read all daily notes for this week (`journals/YYYY/MM-Month/YYYY-MM-DD.md`) — extract "Today I finished" sections
- Check if today's daily note exists; if not, note that it's missing

### Phase 2: Propose wrap-up (1 round)

```
Week WNN wrap-up — Mon DD MMM to Fri DD MMM

Goals review:
1. ✅ / ⚠️ [Goal 1] — [[T-NNN]] done / still in progress
2. ✅ / ⚠️ [Goal 2]
3. ✅ / ⚠️ [Goal 3]

Shipped this week (from daily notes + task completions):
- [[T-NNN - Title]] — marked done Mon
- [[T-NNN - Title]] — marked done Thu
- [other completed items from daily note logs]

Anything else you shipped or want to note?
And: what carries forward to next week?
```

Determine ✅ / ⚠️ per goal by checking if linked tasks are done.

Wait for confirmation, additions, and carry-forward input.

### Phase 3: Write wrap-up (no interaction)

Fill the `## 🏁 Friday Wrap-up` sections in the weekly note:
- `### ✅ Goals achieved` — goals marked ✅
- `### ⚠️ Goals missed / slipped` — goals marked ⚠️ with brief note
- `### 📦 Shipped this week` — full list from daily notes + confirmed additions
- `### ➡️ Carry-forward to next week` — user's carry-forward items
- `### 💡 Reflections` — only add if user provided something; otherwise leave as `- `

Regenerate `Dashboard/Weekly.md`.

### Phase 4: Summary

```
Week WNN closed — Mon DD MMM to Fri DD MMM

Goals: N/M achieved
Shipped: N tasks completed

Carry-forward: N items → will surface in next week's planning
Weekly note: Weeks/YYYY/YYYY-WNN.md

Have a good weekend. 🎉
```

---

## Mode: Status Check

Use this mid-week (Tue–Thu) when the week is already planned.

### Step 1: Fetch (no interaction)

- Read current week note (goals + focus tasks)
- Read `Dashboard/Tasks.md` for focus task statuses
- Read daily notes so far this week for completed items

### Step 2: Report

```
Week WNN status — Day N of 5

Goals progress:
1. 🟢 / 🟡 / 🔴 [Goal 1] — [[T-NNN]] done / in-progress / blocked
2. ...

Shipped so far:
- [[T-NNN - Title]]

Still in progress:
- [[T-NNN - Title]] (P1, due Fri)
```

No writes unless user asks to update something. If they want to adjust goals, update the weekly note and regenerate `Dashboard/Weekly.md`.

---

## Dashboard Regeneration

`Dashboard/Weekly.md` is a generated file showing the current week's goals and progress at a glance.

```markdown
---
tags: Dashboard
week: YYYY-WNN
updated: YYYY-MM-DD
---
# Week WNN · Mon DD MMM – Fri DD MMM

## 🎯 Goals
1. ✅ / 🔄 / ⚠️ [Goal text] → [[T-NNN - Title]]
2. 🔄 [Goal text]
3. 🔄 [Goal text]

## 📋 Focus Tasks
| Task | Status |
|------|--------|
| [[T-NNN - Title]] | in-progress |
| [[T-NNN - Title]] | done ✅ |

---
*Last updated: YYYY-MM-DD*
*Weekly note: [[YYYY-WNN]]*
```

**Goal status icons:**
- `✅` — all linked tasks done (or manually confirmed)
- `🔄` — linked tasks in-progress
- `⚠️` — linked tasks blocked or overdue
- `🔵` — no linked task (free-text goal, status unknown)

Regenerate this dashboard after Monday planning, Friday wrap-up, and any mid-week status updates.

---

## Daily Note Integration

When `/today` runs (morning planning), check if the current week's note exists. If it does:
- Append a link to today's daily note in the `📝 Daily Notes` section of the weekly note under the correct weekday row.

This is a background write — do it silently as part of `/today`'s Phase 4 execution. If the weekly note doesn't exist yet (it's not Monday), skip silently.

**Note**: This integration is a recommendation — `/today` skill should pick this up. The `/week` skill itself always fills daily note links when it reads the weekly note.

---

## Error Recovery

- If `Weeks/YYYY/` doesn't exist: create it silently.
- If last week's note is missing: skip carry-forward with note "No last week note found."
- If daily notes for this week can't be read: continue without them, note what's missing.
- If a linked task file can't be found (may be archived): mark as `done ✅` in the dashboard.
