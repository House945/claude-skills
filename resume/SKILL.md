---
name: resume
description: >-
  Show the board of parked work sessions and resume one on demand. No argument
  = list everything parked (slug, age, next step) and let the user pick, start
  fresh, or prune. Argument = load that slug directly. User-invoked only.
disable-model-invocation: true
---

# resume

Pull a parked note back into context on demand — or just glance at what's on
the desk. Fresh sessions are blank by design; this is how you opt back in.

## With an argument

`$ARGUMENTS` given → target `~/.claude/plans/<slug>-resume.md`.

- Exists → go straight to "Load and orient".
- Missing → say so, then fall through to the board.

## No argument → show the board

```bash
ls -t ~/.claude/plans/*-resume.md 2>/dev/null
```

- None → tell the user nothing is parked, then stop.
- Otherwise build one row per note (newest first):
  - slug (filename minus `-resume.md`)
  - age (mtime, relative — e.g. "2d ago")
  - next step: first non-empty line under the `## What to do next` header
    (fall back to the `## State` branch line if that section is empty)

Present the rows as a numbered list, then offer these choices explicitly:

- `1..N` — resume that task
- `F` — start fresh: do nothing, leave all notes in place (this session stays blank)
- `D` — a task is done: ask which, then delete that one note file

Do NOT guess a choice. Wait for the user. "Start fresh" is a first-class
option, not a fallback.

## Load and orient

Read the chosen note in full. Restate in 3–5 lines: **State**, **Blocked on**,
**What to do next** — so you're oriented before touching code. Then continue.

## Pruning

Delete a note only when the user picks `D` or says the task is done:
`rm ~/.claude/plans/<slug>-resume.md`. Never delete a note you just resumed
unless asked — you may `/clear` and come back to it again.
