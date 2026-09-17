---
name: tasks-everyday
description: Use when helping someone manage their own tasks through the Tasks MCP connector — capturing what they say into the right project, setting up a board for something new, finding what is due, and completing things properly. For personal use rather than agent teams.
---

# Tasks, everyday

You are writing to someone's real task board. It is on their phone. They will
see whatever you do, including the mess.

Two habits cover most of it: **look before you write**, and **fill the fields
that exist** rather than dumping everything into the title.

## Capturing something they said

"Remind me to renew the insurance before the 30th" is a task with a date, not
a sentence in a title.

```
list_projects                  find where it belongs
list_fields { project_id }     get the field ids and option ids
create_task { ... }
```

Put the date in the date field, the urgency in priority, the topic in tags. A
task carrying only a title cannot be filtered, grouped, or found again in two
weeks, which is exactly when they will look for it.

If you are unsure which project something belongs to, ask. Guessing puts their
dentist appointment in a work board.

## Making a new board

Pass **purpose**, in their words. It picks the columns, and it is the
difference between a usable board and an empty list.

```jsonc
create_project {
  name: "Marathon",
  purpose: "training for a first marathon in April, tracking runs and how they felt"
}
```

Known domains — fitness, software, content, study, home, travel — get a field
set designed for them. Anything else gets a sensible general one.

Design the columns yourself only when you know the domain better than the
default will. One field per question:

- **type**: what kind of thing this is, exactly one (Run, Cross-train, Race)
- **tags**: what it is about, several at once (Long, Speed, Recovery)
- **status**: where it is in a workflow, ordered, last one meaning finished
- **priority**: how urgent, only if they genuinely rank things
- **date**: when
- **text**: anything not worth a fixed option list

If a set of labels mixes two questions, split it into two fields. `Kind=Run`
plus `Feel=Hard` can be filtered; one Tags column holding both cannot.

## Completing something

Move it to the last status option:

```jsonc
update_task {
  task_id: "...",
  properties: [{ field_id: "<Status>", value: ["<the last status option>"] }]
}
```

That marks it done, the same way tapping the status does in the app. Moving it
back off that option reopens it.

If a project has no status field, set the state directly:

```jsonc
update_task { task_id: "...", state: "completed" }
```

## Finding things

`search` for free text — "that thing about the boiler".

`filter_tasks` for structured questions — due this week, high priority, tagged
something, nothing assigned. It takes `project_ids` and `conditions`, and
option ids come from `list_fields`.

When they ask "what's on today", filter rather than listing everything and
sorting it yourself.

## Repeating things

A repeat needs a date on the same task, in the same call:

```jsonc
create_task {
  project_id: "...",
  properties: [{ <Title>, "Water the plants" }, { <Date>, 1789200000000000000 }],
  recurrence: { every: "week", on_days: ["wednesday"] }
}
```

`from: "completion"` counts from when they actually finish rather than when it
was scheduled, which is usually what people mean for chores and rarely what
they mean for appointments.

## Timers

`start_timer` shows a live timer on their lock screen; `stop_timer` ends it
and records the duration. Useful when they ask you to time something. One per
task.

Do not start timers they did not ask for. It is their lock screen.

## Things that cannot be undone

Only tasks can be deleted. **A project or field, once created, stays.** Do not
create either to try something out, and do not add a field speculatively. Ask
first if you are unsure.

## Values, briefly

Option-backed fields take arrays of ids, never names. Dates are nanosecond
epoch numbers. Writing a multi-value field replaces what was there, so read it
first if you mean to add rather than replace.

Icons and colours come from a fixed set the app can render. Leave them out and
a sensible one is chosen from the name.
