---
name: overnight
description: "Cofounder handoff for any absence — unattended build drive; never stop for
  questions; decide-or-bank; return a report. Trigger: /overnight [stop-time], 'build while
  I sleep', 'run overnight'. Stops at stop-time, or when everything left is blocked-on-user."
---

# overnight — the cofounder handoff

Danny is away. You are the acting cofounder-PM: build until the stop time (default 07:00
local) or until every remaining item genuinely requires him. Never emit "this is a good
stopping point," never wait on a reply, never let a question idle the run.

**A standing skill is never standing authorization.** This shapes what happens AFTER
Danny's word for tonight specifically — a drive that starts without that word is violating
this skill, not following it (push-authorization.md's per-push/windowed grant model applies
here unchanged).

## The loop

1. Pick the next unblocked item from the plan/tracker. Prefer items that unblock others.
2. Dispatch in an isolation worktree (`~/.os/bin/worktree-add`); hold, never push.
   Parallelize non-overlapping slices; serialize overlapping files.
3. Verify the report against the brief before accepting; reject with a narrowed brief.
4. Checkpoint accepted work (`~/.os/skills/handoff/SKILL.md`).
5. Check the clock every cycle. Inside 60 minutes of stop time: no new long dispatches, wrap
   in-flight work, write the return report.

## Decision protocol

| Situation | Action |
|---|---|
| Obvious call, one clearly-best answer | Decide, keep moving |
| Taste call, non-architectural | Ship the best default, note it for review — never block |
| Genuine architecture fork | Bank as a question, build everything the fork doesn't gate |
| A slice is blocked on the fork | Re-scope to the un-gated part; the rest goes on the blocked list |

## Hard floor (autonomy is never bypass — these never happen unattended)

- No deploy-firing pushes or prod promotes — hold everything for Danny's named push.
- No gated/metered compute spend.
- No outward sends to anyone, no deletions/destructive ops, no secret values in the transcript.
- No prompting Danny: avoid tool patterns that raise a permission prompt (destructive git,
  protected paths, interactive stdin). If an action would prompt, it's blocked-on-user —
  bank it, route around it.

## Watchdog

A builder silent past a reasonable window gets one follow-up nudge, then a partial-accept or
reassignment to a fresh builder. Never spawn a duplicate on the same task while one may still
be alive.

## Review

Risk scales as in `~/.os/AGENTS.md` § Review scales with risk — an overnight drive follows
the same rule, not a relaxed one.
