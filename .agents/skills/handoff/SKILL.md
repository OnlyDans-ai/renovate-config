---
name: handoff
description: Writes or resumes docs/HANDOFF.md, the record of unfinished work that carries a task across sessions,
  harnesses, compaction or a quota stop. Use at milestones in sustained work, before parking a task or switching hosts,
  at session close, and when a session continues earlier work.
---

# Handoff

`docs/HANDOFF.md` holds what is unfinished. Approved design lives in `plans/`, durable lessons in `.agents/memory/`.
`os handoff` writes the template if it is missing, else prints it with the git log since its last update.

## Writing

Checkpoint at milestones without being asked; one-step work needs none.

- **State**: what is true now, with its evidence (commits, review unit ids, logs), so a fresh session re-derives nothing.
- **Next action**: the single next step.
- **Open decisions**: what waits on Danny.
- **Constraints**: anything scoped or time-boxed that must not be dropped.

A handoff records work; it does not certify it done or close a review.

At close, if `os status` prints a `memory:` line, consolidate: merge overlapping topic files, drop what the code now
says, keep MEMORY.md at one line per file. A lesson that lives only in a plan or in the harness's private store
becomes a topic file.

## Resuming

Continue the unfinished task unless Danny gives a new objective. Read the handoff, then the plan it names, and
reconcile moved revisions and dirty work before repeating anything. A new host or session grants no new authority.
Context size is Danny's to manage; keep working.
