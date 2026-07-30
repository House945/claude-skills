---
name: park
description: >-
  Park the current work session to a plan file so it survives a /clear or a
  fresh session. Does NOT auto-load into future sessions — pull it back on
  demand with /resume. User-invoked only.
disable-model-invocation: true
---

# park

Write everything needed to resume the CURRENT session later into a plan file.
Nothing goes into auto-loaded memory, so fresh sessions stay blank — you pull
the note back yourself with `/resume` when you want it.

Doubles as a controlled compaction: to keep working the same task on a fresh
buffer, run `/park <slug>`, then `/clear`, then `/resume <slug>`. Unlike
`/compact`, YOU decide what survives — exactly the sections in the note, no
lossy model-chosen summary.

## Argument

`$ARGUMENTS` — optional slug (e.g. `vc-funds`). If empty, derive from the git
branch: `git branch --show-current`, take the last path segment, lowercase,
spaces/slashes → `-`. No branch and no arg → STOP and ask; don't invent one.
Call it `<slug>`.

## Step 1 — Ground-truth state (Bash, don't guess)

```bash
git branch --show-current
git log -1 --format='%H %s'
git status --short
git rev-list --left-right --count @{upstream}...HEAD 2>/dev/null || echo "no upstream"
git log --oneline @{upstream}..HEAD 2>/dev/null || git log --oneline -10
```

Record: branch, tip SHA + subject, pushed (ahead/behind), tree clean/dirty,
commit trail, any "commit X largely undone by Y" gotcha, and any MR/PR you
already know from THIS session (don't go hunting for one).

## Step 2 — Draft the note

Ground every line in Step 1 + what you learned in THIS session. Don't pad; drop
sections that don't apply rather than writing "N/A".

Rot-resistance:

- Reference commits by SHA + branch, never "the last commit".
- Prefer greppable markers over counts: `grep -n '[POC]'`, not "12 POC stubs".
- If a commit is undone by a later one, name both SHAs so nobody re-derives it.

Skeleton:

```markdown
# Resume: <slug>

## State

- Branch: <branch> | MR/PR: <id or "none">
- Tip: <sha> "<subject>" | Pushed: <yes/no + ahead/behind> | Tree: <clean/dirty>
- Commit trail: <oneline list or a marker if long>
- History gotchas: <e.g. "abc123 largely undone by def456">

## Blocked on

<what blocks, and who owns the unblock>

## What to do next

<be honest — if it's a clean stopping point, write "Nothing unless asked">

## Concrete next steps (only if real)

<numbered, each with a greppable anchor where possible>

## Contract to hand off (if someone else continues)

<endpoints / assumptions / ordering / must-get-right / warnings>

## Decisions NOT to reopen

<settled calls + one-line rationale each>

## Parked — do NOT start without asking

<deferred things>

## Verification notes

<login gotchas, seeding dead ends, how to actually check it works>

## Traps

<the specific ways this gets done wrong>

## Working preferences (this task)

<complete-file replacements not diffs; don't push without asking; etc.>
```

## Step 3 — Write the note

Full replacement at `~/.claude/plans/<slug>-resume.md`. Overwrite if it exists —
the note is a snapshot, not a log.

## Step 4 — Self-check (fail loud)

```bash
wc -l < ~/.claude/plans/<slug>-resume.md
```

If the write failed, say so and fix it before finishing.

## Step 5 — Report

3–5 lines: slug, note path + line count, tree state. Then:

> To resume later: `/resume` (shows the board) or `/resume <slug>` (direct).
> Fresh sessions stay blank until you ask.
