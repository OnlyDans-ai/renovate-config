---
name: wizard
description: Generates an interactive bash wizard that walks Danny through steps only a human can take. Use when a
  task needs him to provision infrastructure, set up credentials or CI secrets, click through a third-party dashboard,
  or run a one-off migration or cutover. Not for steps the agent can do itself.
---

# Wizard

A **wizard** is a bash script that walks Danny, stage by stage, through a manual procedure
that is tedious by hand and tedious to re-explain every time. It opens each URL, says
exactly what to click, captures each value, writes it where it belongs, confirms before
anything irreversible, and shows how many stages remain.

This is the mechanism behind two boundaries in AGENTS.md: anything handed to Danny becomes
`bash /tmp/wizard/<action>.sh`, and secrets are provisioned 1Password-first, never pasted
into chat. Reach for it only when a task hits a wall only Danny can pass — if the agent can
do it, the agent does it.

The UX is already solved by [template.sh](template.sh). **Author only the STAGES section at
the bottom** — the library above it is identical in every wizard; never hand-edit it. A
wizard is ephemeral by default: written to `/tmp/wizard/<action>.sh`, deleted when the job is
done. Commit it to `scripts/` only when Danny wants a repeatable setup path in the repo.

## Process

1. **Scope the procedure.** Read the repo first, never ask cold: `.env.example`, the variable
   names in `.env` (names only, never values), README, `docker-compose*`, framework config, and
   every `secrets.*`/`vars.*` reference in CI workflows is a value the wizard must produce.
2. **Map each stage's journey.** For each captured value: which URL, what to click, where the
   value appears, where it's written (`.env`, a GitHub secret, 1Password, or nowhere for a
   pure action), and whether it's secret. Where you don't know the current UI, say so — never
   invent steps that may not exist.
3. **Author the stages.** Copy `template.sh`, replace the example with one `stage` per step in
   dependency order, set `TOTAL_STAGES` to match. Secrets are 1Password-first: create a
   PASTE-ME placeholder item, hand Danny the UI path to fill it, then provision with
   `op-sa.sh <account> read "op://…/credential" | gh secret set NAME` (or the equivalent
   sink) — the value never transits chat or this script's own output.
4. **Hand it back:** the ordered stage names, then `bash /tmp/wizard/<action>.sh`. Danny may add, drop or
   reorder stages before running it.
