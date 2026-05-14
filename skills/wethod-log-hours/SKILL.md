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

### Step 1 — Ask user for date range

Ask the user which dates they want to log hours for:

**English:** "Which dates do you want to log hours for? (e.g. today, this week, or a range like 12/05 - 14/05)"  
**Spanish:** "¿Para qué fechas quieres imputar horas? (ej. hoy, esta semana, o un rango como 12/05 - 14/05)"

Resolve the answer to a list of individual working days (Mon–Fri only, skip weekends). For each day compute:
- Date (YYYY-MM-DD)
- Monday of that day's week (YYYY-MM-DD) → `date` param for `create_timesheet`
- Day of week abbreviation (`mon` / `tue` / `wed` / `thu` / `fri`)

### Step 2 — Check available hours for the range

Call `check_timesheet_status` with the `person_id` from Step 0. It returns total/missing hours per day for the current week.

For each day in the range, extract missing hours (max 8h/day). Skip days that already have 8h logged (inform user which days are skipped).

If all days in range already fully logged → inform user, stop.

### Step 3 — Find last used project

Call `list_timesheets` with the `person_id` from Step 0, `limit: 1`. This returns the most recent timesheet entry, including its `project_id`.

Then call `get_project` with that `id` to get the project name.

### Step 4 — Confirm project and range with user

Show recap (in detected language):

**English:**
> Last project: **[Project Name]** (id: XXXX)  
> Log hours for the following days in this project?  
> - [weekday] DD/MM/YYYY → Xh  
> - [weekday] DD/MM/YYYY → Xh  
> ...

**Spanish:**
> Último proyecto: **[Project Name]** (id: XXXX)  
> ¿Registrar horas en los siguientes días en este proyecto?  
> - [día] DD/MM/YYYY → Xh  
> - [día] DD/MM/YYYY → Xh  
> ...

Wait for user response.

### Step 5a — User confirms → log hours for all days

For each day in the range, call `create_timesheet`:
```
person_id: [from Step 0]
date: [Monday of that day's week]
day: [weekday abbreviation for that day]
hours: [missing hours for that day, max 8]
project_id: [last project id]
confirm: true
```

Call sequentially, one per day. Report progress after each call.

### Step 5b — User declines → show project selector

Call `list_projects` with `limit: 100`. Show the list grouped or filtered by probability (100% first). Present as a numbered/named list for the user to pick from.

Once user selects a project, log hours using `create_timesheet` for each day in the range with the chosen `project_id`.

## Notes

- `date` in `create_timesheet` must always be the **Monday** of the target week, not the actual day's date.
- `day` is the actual weekday of each day in the range.
- `hours` = missing hours per day from `check_timesheet_status` (never exceed 8).
- Skip weekends when resolving date ranges.
- `check_timesheet_status` only returns the current week — if range spans multiple weeks, call it once per week.
- Always show a full recap before calling `create_timesheet` with `confirm: true`.
