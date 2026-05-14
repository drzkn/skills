---
name: wethod-log-hours
description: Log timesheet hours in Wethod for the current day. Checks the last used project, confirms with the user, then logs maximum available hours. Falls back to a project selector if the user wants a different project. Use when user says "registrar horas", "imputar horas", "log hours", "fichar", "wethod horas", or similar.
---

# Wethod — Log Hours for Today

**MCP server**: `user-wethod`

> The `person_id` is provided by the MCP server context — read it from there. Do NOT hardcode it.

## Workflow

### Step 0 — Get person_id from MCP server

Read the `person_id` injected by the `user-wethod` MCP server (available in the server use instructions / context). Use that value for all subsequent calls.

### Step 1 — Compute today's date info

Determine:
- Today's date (YYYY-MM-DD)
- Monday of the current week (YYYY-MM-DD) → this is the `date` param for `create_timesheet`
- Day of week abbreviation (`mon` / `tue` / `wed` / `thu` / `fri`)

### Step 2 — Check available hours for today

Call `check_timesheet_status` with the `person_id` from Step 0. It returns total/missing hours per day for the current week. Extract the missing hours for today — that is the maximum to log.

If today already has 8h logged → inform user, stop.

### Step 3 — Find last used project

Call `list_timesheets` with the `person_id` from Step 0, `limit: 1`. This returns the most recent timesheet entry, including its `project_id`.

Then call `get_project` with that `id` to get the project name.

### Step 4 — Confirm project with user

Show:
> Last project: **[Project Name]** (id: XXXX)  
> Log Xh for today ([weekday] DD/MM/YYYY) in this project?

Wait for user response.

### Step 5a — User confirms → log hours

Call `create_timesheet`:
```
person_id: [from Step 0]
date: [Monday of current week]
day: [today's weekday abbreviation]
hours: [missing hours for today, max 8]
project_id: [last project id]
confirm: true
```

### Step 5b — User declines → show project selector

Call `list_projects` with `limit: 100`. Show the list grouped or filtered by probability (100% first). Present as a numbered/named list for the user to pick from.

Once user selects a project, log hours using `create_timesheet` with same params but the chosen `project_id`.

## Notes

- `date` in `create_timesheet` must always be the **Monday** of the target week, not today's date.
- `day` is the actual weekday of today.
- `hours` = missing hours for today from `check_timesheet_status` (never exceed 8).
- Always show a recap before calling `create_timesheet` with `confirm: true`.
