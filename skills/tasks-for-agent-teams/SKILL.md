---
name: tasks-for-agent-teams
description: Use when several agents and a person share one Tasks board — a builder, a reviewer, a researcher, a human orchestrator. Covers claiming work without collisions, handing off with enough context, marking who did what, and knowing when to stop and ask.
---

# Tasks for agent teams

One board, several workers, no coordination protocol between them. Everything
you know about what the others are doing, you know because you read the board.

This works because the board is the only shared state, and it is the same
state the person sees on their phone. There is no separate agent channel to
fall out of sync with.

## The rule that prevents collisions

**Read immediately before you write.** Not at the start of your session, not
from something you cached. Another agent may have claimed the task in the
seconds since.

```
get_task { task_id }        what is its status right now
```

If it has moved to an in-progress stage and you did not move it there, leave
it alone and pick something else. There is no locking. The convention is the
lock.

## Claiming

Move it in-progress **before** starting, not after finishing. The whole point
is that another agent reading the board a moment later sees it is taken.

```jsonc
update_task {
  task_id: "...",
  properties: [
    { field_id: "<Status>", value: ["<In Progress>"] },
    { field_id: "<Worker>", value: ["<your worker option>"] }
  ]
}
```

## Saying who you are

Nothing tells the board which agent you are. An OAuth connection acts as the
person who authorised it, so every agent's work arrives looking identical and
indistinguishable from the human's.

If the board has a `Worker` or `Agent` field, **set it on every task you
touch**. It is the only thing making the board legible when three agents have
been through it.

If it does not have one, ask before adding it. A field cannot be deleted once
created.

Be honest in it. The point is a record of who did what, and a board where
everything claims to be one agent is worse than no field at all.

## Handing off

A handoff note is the message the next worker gets. They have no memory of
you, no access to your reasoning, and no way to ask.

Bad:

> Done. Looks good.

Good:

> Indexes are on properties and tasks, p50 dropped 135ms to 21ms. The tasks
> endpoint did not improve and I do not know why yet; it is a write path, so
> the index was probably never going to help it. Next: measure whether the
> per-task query count is the cost. Do not re-run the migration, it is live.

Write what you found, what is unresolved, and what you would do next.
Especially write what you were **wrong** about, because the next worker will
otherwise repeat it.

## Reviewing someone else's work

When you pick up something in a review stage:

Read the handoff note first. Check the claim, not the summary — if it says
tests pass, run them. If you disagree, do not silently redo the work: move it
back with a note saying what you found. The other agent may have had a reason
you cannot see.

Pass it by moving it forward and noting what you checked. "Reviewed" tells the
next person nothing. "Ran the suite, 276 pass. Confirmed the migration is
idempotent by running it twice" does.

## Knowing when to stop

Some things are not yours to decide. A schema change, a production deploy,
anything touching money or user data, anything where being wrong is expensive
and you are unsure.

Put those in a blocked stage with a note explaining the decision needed and
the options as you see them. Do not leave them in progress: another agent will
think you are still on it and the person will not know they are being waited
on.

```jsonc
update_task {
  task_id: "...",
  properties: [
    { field_id: "<Status>", value: ["<Blocked>"] },
    { field_id: "<Handoff Note>", value: "Needs a human: the fix changes behaviour the iOS client shares. Options are (a) document it, (b) change it and update the client. I would do (a) first." }
  ]
}
```

## Finishing

Two fields, always:

```jsonc
update_task {
  task_id: "...",
  state: "completed",
  properties: [{ field_id: "<Status>", value: ["<the last status option>"] }]
}
```

Status alone does not complete a task. A board full of Done cards that are all
still pending is the normal result of one agent getting this wrong, and it is
invisible until someone filters by state.

## Showing the person you are working

```
start_timer { task_id }   ... work ...   stop_timer { task_id }
```

A running timer appears live on their lock screen. With several agents going,
it is how a person tells "three things are happening" from "everything is
stuck". One timer per task.

`list_running_timers` shows what is in flight across the whole board, which is
the closest thing to a standup.

## Practical limits

Writing a multi-value field replaces it, so read before you write or you will
drop another agent's tags.

Only tasks can be deleted. No project deletion, no field deletion. Treat
creating either as a decision, not an experiment.

Option ids belong to one field. An id from another column is refused rather
than quietly stored.
