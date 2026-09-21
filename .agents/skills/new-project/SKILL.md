---
name: new-project
description: Project inception + OS bootstrap — vision/PRD interview, evidenced stack
  recommendation, repo scaffolded to standard OS posture. Use when starting ANY new
  project — "new project", "set up X", "project setup", "start X".
---

# new-project — Inception → PRD → Stack → Scaffold

Goal: a new project is born with its vision captured, its requirements written, its stack
decided on evidence, and its harness conformant — one session instead of an afternoon of
retrofitting. Two phases, hard gate between them: **Phase A** produces ratified artifacts;
**Phase B** is mechanical and runs only after Danny ratifies the stack.

## Phase A — Inception

1. **Interview** (2-3 rounds, batched — never one question at a time): (1) vision — what,
   for whom, why now, what success looks like; (2) product shape — surfaces, integrations,
   data sensitivity, scale; (3) constraints — timeline, budget, existing infra to reuse,
   what's explicitly out. Capture Danny's verbatim phrasing.
2. **Write `docs/VISION.md`** (narrative, success criteria) and **`docs/PRD.md`**
   (jobs-to-be-done, requirements, non-goals, milestones v0->v1, open questions). Present
   both; iterate until ratified. No scaffold before a ratified PRD.
3. **Stack recommendation** — evidence, then opinion, never vibes. Research the moving parts
   (web search, package registries) where the space has moved recently. Deliver:
   `Recommendation: {stack}. Why: {reason traced to a PRD requirement}. Alternatives: {option
   -> why not}.` Danny ratifies or overrides.

## Phase B — Scaffold (mechanical, profile-scaled)

The interview determines the profile; ceremony scales to it:

| Profile | Gets |
|---|---|
| **full-product** | Everything below |
| **website** | Repo init + CLAUDE.md + secret posture + `.gitguardian.yaml`; skips tribunal/observability bindings unless the site has real logic |
| **non-code** | Repo init + secret posture only |

1. `git init` + first commit with `.gitignore` never-commit classes (`.env*`, `credentials/`,
   `*.token`); `gh repo create --private` + push; confirm secret-scanning covers the new repo.
2. `AGENTS.md` core block (copy between `<!-- os:core:start/end -->` from `~/.os/AGENTS.md`)
   + a project block from `~/.os/templates/AGENTS.project.md` filled with stack/commands/deploy
   facts from VISION.md and the ratified stack. `CLAUDE.md` = `@AGENTS.md`.
3. `.agents/skills/` (copy of `~/.os/skills`), `.claude/skills` symlink -> `../.agents/skills`.
4. `.agents/memory/MEMORY.md`, `.githooks/pre-commit` + `pre-push` (from `~/.os/templates/
   githooks/`), `git config core.hooksPath .githooks`.
5. `docs/HANDOFF.md` from the template, `plans/`, `.claude/settings.json` (permissions,
   `plansDirectory`, `autoMemoryDirectory`), `.agents/deploy-branches` (empty).

Idempotent: never overwrites an existing project block, handoff, or memory.
