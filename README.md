# Tasks MCP skills

Skills for working with [Tasks](https://thetasks.app) through its MCP
connector — a shared task board that agents write to and people read on their
phone.

The connector works without any of this. These add the parts a tool schema
cannot express: which order to do things in, what a good handoff note looks
like, and the handful of behaviours that surprise people.

## Connect

```
https://api.thetasks.app/mcp
```

No API key. The connector registers itself and takes you through sign-in.

**Claude Code**

```
claude mcp add --transport http tasks https://api.thetasks.app/mcp
```

**Claude (web, desktop, mobile)** — Settings → Connectors → Add custom
connector, paste the URL.

**ChatGPT** — Settings → Connectors → add an MCP server with the URL.

**Codex** — add it as an MCP server in your config with the same URL.

The first call returns a sign-in link. After that the connection persists.

## The skills

| Skill | For |
|---|---|
| [`tasks-for-coding-agents`](skills/tasks-for-coding-agents/SKILL.md) | An agent working in a repo: plan work as a board, track it while doing it, leave a trail |
| [`tasks-for-agent-teams`](skills/tasks-for-agent-teams/SKILL.md) | Several agents and a person on one board: claiming, handoffs, review, knowing when to stop |
| [`tasks-everyday`](skills/tasks-everyday/SKILL.md) | Personal use: capture, boards, finding things, recurring chores |

Take one. They overlap deliberately, because each is meant to be read alone.

## Installing

**Claude Code** — copy a skill folder into `.claude/skills/` in your project,
or `~/.claude/skills/` for every project:

```bash
git clone https://github.com/Msquare-Labs/tasks-mcp-skills
cp -r tasks-mcp-skills/skills/tasks-for-coding-agents ~/.claude/skills/
```

**Anywhere else** — paste the contents of a `SKILL.md` into your system prompt,
project instructions, or whatever your client calls that. They are plain
Markdown and assume nothing about the host.

## Three things worth knowing before you start

**Moving a task to the last status option completes it**, and moving it off
again reopens it. Setting `state` directly still works and wins when a task is
finished but its column belongs elsewhere.

**Only tasks can be deleted.** Projects and fields, once created, stay. Create
them deliberately rather than to try something out.

**Writing a multi-value field replaces it.** Sending one tag drops the others.
Read the current values first, or use `add_subtasks` / `remove_subtasks` for
subtasks, which exist for this reason.

The server tells clients all of this on connect, so a capable model will often
get it right without a skill. The skills are for the parts that are judgement
rather than rules.

## Licence

MIT. Copy, adapt, ship your own.
