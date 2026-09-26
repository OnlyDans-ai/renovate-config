---
name: new-project
description: Starts a new project from nothing - vision and PRD interview, an evidenced stack recommendation, then a
  repo scaffolded to the OS standard. Use when Danny says "new project", "start X", "set up X" for something that has
  no repo yet, or "project setup".
---

# New project

Two phases with a gate between them: nothing is scaffolded until Danny ratifies the PRD and the stack.

## Phase A: inception

1. Interview in two or three batched rounds: the vision (what, for whom, why now, what success looks like), the
   product shape (surfaces, integrations, data sensitivity, scale), and the constraints (timeline, budget, infra to
   reuse, what is out). Keep Danny's own phrasing.
2. Write `docs/VISION.md` (narrative, success criteria) and `docs/PRD.md` (jobs to be done, requirements, non-goals,
   milestones v0 to v1, open questions), and iterate until he ratifies them.
3. Recommend the stack from current sources, as the consultant skill does, each choice traced to a PRD requirement.

## Phase B: scaffold

The profile decides the ceremony: a **product** gets all of it; a **website** skips observability unless it has
real logic; **non-code** gets the repo and the secret posture only. Review follows the change's risk (AGENTS.md),
whatever the profile.

1. `git init`, a `.gitignore` covering `.env*`, `credentials/` and `*.token`, a first commit, and
   `gh repo create --private`. Confirm secret scanning covers the new repo.
2. `os init .` lays down the OS: AGENTS.md core block, CLAUDE.md pointer, skills, memory, handoff, git hooks,
   `.claude/settings.json`, `.agents/deploy-branches`, MCP files. It never overwrites what exists.
3. Fill the AGENTS.md project section (stack, run, test, deploy) from the PRD and the ratified stack.
4. Set up `.agents/check` and the CI workflow as the test-tiers skill describes.
