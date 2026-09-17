---
name: tasks-for-coding-agents
description: Use when working in a codebase with the Tasks MCP connector available, to plan work as a board, track what you are doing while you do it, and leave a trail the human can read on their phone. Covers claiming work, timers, handoff notes and finishing tasks correctly.
---

# Tasks for coding agents

You keep no memory between sessions. The board does. Anything you write here
survives you, and the person you are working for can read it on their phone
while you are still running.

That is the whole reason to use it: not bookkeeping, but leaving state
somewhere a human and the next session can both reach.

## Before you touch anything

```
list_projects                  which boards exist
list_fields  { project_id }    the field ids and option ids you need to write
```

Field sets differ per project. You cannot write a value without the ids, and
the ids are not guessable. Do this once at the start rather than per task.

## Finishing a task takes two things

This is the mistake that makes a board useless, so learn it first:

```jsonc
update_task {
  task_id: "...",
  state: "completed",                              // the task is done
  properties: [{ field_id: "<Status>", value: ["<Done>"] }]   // the column says so
}
```

**Setting Status to Done does not complete the task.** They are separate. A
board where every card reads Done and every task is still pending is what
happens when an agent sets one and not the other. Set both, and set both back
when reopening.

## The loop

Work in a cycle the human can follow without asking you anything.

**Claim it.** Move the task to an in-progress stage before you start. Another
agent reading the board then knows not to take it. Read the task first: the
status may have moved since you last looked.

**Start a timer.**

```
start_timer { task_id }
```

This puts a live timer on the person's lock screen. It is the difference
between "an agent is working" and silence. Start it when you begin real work,
not around a single tool call. One timer per task; starting a second is
refused.

**Do the work.** Normally, in the repo.

**Stop the timer and leave a note.**

```
stop_timer  { task_id }
update_task { task_id, properties: [{ <Handoff Note>, "what you found, what is left, what you would do next" }] }
```

The note is the part that matters. Write it for whoever picks this up with no
memory of you, which may well be you tomorrow. "Fixed" is useless. "Indexes
added, p50 went 135ms to 21ms, the tasks endpoint did not move and I do not
yet know why" is what the next session needs.

**Finish it,** with both fields as above.

## Turning a repo into a board

When asked to plan work, create the project with a purpose rather than a bare
name. The purpose picks the columns.

```jsonc
create_project {
  name: "API v2",
  purpose: "rewriting the sync endpoints, one task per endpoint with review before merge",
  status_options: ["Backlog", "In Progress", "Needs Review", "Blocked", "Shipped"]
}
```

The last status option is treated as finished. Put the terminal stage last and
nothing else after it.

Then one task per unit of work someone could pick up alone. If a task cannot
be finished without another one landing first, make it a subtask rather than a
separate row:

```
update_task { task_id: "<parent>", add_subtasks: ["<child>"] }
update_task { task_id: "<parent>", remove_subtasks: ["<child>"] }
```

Removing a subtask does not delete it. It becomes an ordinary task again.

## Writing values

Everything option-backed takes an **array of ids**, never a name:

```jsonc
{ field_id: "<Tags>", value: ["A3D3...", "9E92..."] }   // right
{ field_id: "<Tags>", value: "backend" }                // refused
```

Dates are **nanosecond epoch numbers**. Multiply seconds by 1e9. A string is
refused.

Writing a multi-value field **replaces** it. Sending one tag drops the rest.
Read the current ids with `get_task` first and send the whole list.
`add_subtasks` and `remove_subtasks` are the exception, and exist for exactly
this reason.

## What you cannot undo

Only tasks can be deleted. **There is no way to delete a project or a field.**
Create both deliberately: a throwaway project to try something out is
permanent, and a field added to see what happens is a column the person lives
with.

Rename, reorder and hide are available instead.

## Recurring work

A repeat needs a date on the same task in the same call:

```jsonc
create_task {
  project_id: "...",
  properties: [{ <Title>, "Weekly dependency audit" }, { <Date>, 1789200000000000000 }],
  recurrence: { every: "week", on_days: ["monday"] }
}
```

A rule with no date is refused, because there is nothing to repeat from.

## Working next to other agents

Assume you are not alone. Another agent or the person may be on the same board
right now.

Read before you write. Claim before you start. Note before you stop. If
something is blocked on a decision only a human can make, say so in the note
and move it to a blocked stage rather than leaving it in progress, so nobody
waits on you.
