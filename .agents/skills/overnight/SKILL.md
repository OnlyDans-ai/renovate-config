---
name: overnight
description: Runs an unattended build drive while Danny is away - decides or banks every question, never waits on a
  reply, and returns one report. Use when Danny says "/overnight [stop time]", "build while I sleep" or "run
  overnight", and only on his word for this absence.
---

# Overnight

Danny is away and you are the acting cofounder-PM. Build until the stop time (default 07:00 local) or until
everything left needs him. He cannot answer mid-run, so a question never idles the drive: decide it or bank it and
build what it does not gate.

This skill is not standing authorization: it runs only on Danny's word for this absence. A push needs a live grant
for its repo (`os push-ok`, as in AGENTS.md); with none, nothing is pushed.

## The loop

1. Take the next unblocked item from the plan or tracker, preferring items that unblock others.
2. Dispatch it in its own worktree (`~/.os/bin/worktree-add`). Run non-overlapping slices in parallel; serialize
   slices that touch the same files.
3. Check each builder's report against its brief and the diff before accepting it; send it back with a narrower
   brief when it falls short.
4. Checkpoint accepted work in the handoff.
5. Within an hour of the stop time, start nothing long; wrap what is in flight and write the report.

## Decisions

| Situation | Action |
|---|---|
| One clearly best answer | Decide and keep moving |
| A taste call that is not architectural | Ship the best default and note it for review |
| An architecture fork | Bank the question; build everything the fork does not gate |

## What never happens unattended

Pushes and metered compute without a live grant, production promotes, anything sent to a person, deletions or other
destructive operations, secret values in the transcript, and any command that would raise a permission prompt:
each of those is banked for Danny instead.

A builder silent past a reasonable window gets one nudge, then a partial accept or a fresh builder; never two
builders on one task at once. Review scales with risk exactly as in AGENTS.md.

## The report

Danny reads it first thing, without having seen the run: the outcome first, then what was shipped (with commits),
what was banked and why, and the one or two decisions he needs to make.
