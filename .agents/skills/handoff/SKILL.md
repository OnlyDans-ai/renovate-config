---
name: handoff
description: Capture or resume active work across sessions, hosts, compaction, or quota
  interruption, using the project's docs/HANDOFF.md and its plan/tracker references. Use at
  meaningful milestones during sustained work and before switching hosts or parking a task.
---

# Handoff

`docs/HANDOFF.md` records current state, the next action, evidence, and scoped constraints.
Plans (`plans/*.md`) own approved design; a small task needs neither a plan nor a tracker.
Durable memory lives in `.agents/memory/`; the handoff carries unfinished work — the two are
not the same thing. `os handoff` writes the template if missing, otherwise prints it and the
git log since its last update so the PM fills it.

## When to checkpoint

Proactively, at meaningful milestones during sustained work — Danny need not request it.
Keep the task active while working, paused when parked, and completed when its acceptance
conditions are met. Ordinary one-step work needs no ceremony. A capture does not certify
completion or waive outstanding review; abrupt interruption can lose unrecorded state.

Write:
- **State** — what's true right now, in enough detail that a fresh session doesn't have to
  re-derive it.
- **Next action** — the single next concrete step, not a backlog.
- **Open decisions** — anything still waiting on Danny's word.
- **Constraints** — anything scoped or time-boxed that a fresh session must not silently drop.

## Resuming

On a fresh session: continue the selected unfinished task unless Danny supplies a different
objective. Read `docs/HANDOFF.md` first, then its referenced plan/tracker. Reconcile changed
revisions and dirty work before repeating anything — a report is not evidence; verify claims
yourself. A host switch grants no new authority and resets no completed work.

## Context lifetime

Continue useful authorized work in the current session; checkpoint at natural phase
boundaries without pausing work or changing native context. Danny owns interactive
compaction, new-session, model, and effort controls — this skill never blocks or nags about
context size; a session cannot see its own token count reliably enough to police it, and a
2026-model manages its own context well enough to say when it's getting tight.
