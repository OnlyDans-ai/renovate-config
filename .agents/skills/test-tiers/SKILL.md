---
name: test-tiers
description: The OS standard for how a repo's tests and CI are split into tiers so an agent
  gets fast feedback and pushes don't burn compute. Use when setting up or changing a repo's
  `.agents/check`, test runner config or `.github/workflows/`, when a check or CI run takes
  longer than its tier's budget, or when auditing a repo's test and CI cost.
---

# Test tiers

Every test belongs to one tier, and each tier has a budget. A suite that blows its budget is a
defect to fix, like a failing test. The cost is agent time, CI minutes (GitHub Actions,
Blacksmith) and billed services (Neon branches, hosted APIs).

| tier | runs when | budget | what |
|---|---|---|---|
| T0 commit | every commit (pre-commit) | seconds | format, lint, the secret scan |
| T1 inner loop | while an agent works | under 2 min | only the tests the change affects (pytest-testmon, `vitest --changed`, Nx/Turbo affected) |
| T2 gate | `os review` gate 0, before a push to a deploy branch, CI on that push | 10–15 min | the whole fast suite, in parallel; the OS keeps a PASS for the exact tree, so an unchanged tree never reruns it |
| T3 nightly / on demand | schedule or `workflow_dispatch` | as long as it takes | live infra, perf, e2e against real services |

## Rules

1. **Parallel.** A suite that can run in parallel does: `pytest -n auto` (xdist),
   vitest/jest workers, go test's default. Tests that can't share a process are marked (`serial`)
   and run on a second line of their own.
2. **No real database or billed service in T1/T2.** Fast tests use a local Postgres (docker,
   `pg_tmp`, a service container in CI), an in-memory store or fakes. A test that needs a Neon
   branch or a hosted API is T3 and carries a marker that T2 excludes.
3. **The check is declared once.** `.agents/check` is the T2 gate. CI runs `.agents/check`
   itself, or exactly its lines. Two lists drift apart.
4. **CI runs only on what it gates.** Pushes to deploy branches and PRs into them, not every
   lane branch. `on: push` without a branch filter plus `pull_request` runs twice per PR
   commit. Add path filters to monorepo jobs, and `concurrency: { group: <workflow>-<ref>,
   cancel-in-progress: true }` so a newer push cancels the older run.
5. **Every schedule is justified.** An hourly or daily cron says in a comment what it catches
   that a push-triggered run doesn't. Uptime pings belong to an uptime service, not Actions.
6. **Cache.** Dependency caches (`setup-node` `cache:`, `setup-uv` `enable-cache`, pip cache),
   and build caches where the tool has one (Turbo, Next, Docker layer cache).
7. **Know the slowest tests.** `--durations=25` (pytest) or the runner's report is on, and the
   top offenders are fixed or moved to T3.
8. **Size runners to the job.** A large runner (Blacksmith 8/16 vCPU) only where the job uses
   the cores. A lint job on 16 vCPU costs the same per minute as a parallel suite.

## The CI shape every repo uses

Every repo that deploys has one CI workflow, and it runs the gate itself. `.agents/check` holds everything that
would break a deploy (lint, typecheck, tests, and the build when the deploy builds), so CI and `os review` can't
disagree. It is one bash script, starting with `set -euo pipefail` and a `cd` to the repo top; both run it whole with
`bash .agents/check`, so the script's own `set` line decides how it stops.

```yaml
name: ci
on:
  push:
    branches: [main, preview]          # the repo's deploy branches, as in .agents/deploy-branches
  pull_request:
    branches: [main, preview]
  workflow_dispatch:
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
jobs:
  check:                               # keep the job name a branch protection rule already requires
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Docs-only change?
        id: scope
        env:
          BASE: ${{ github.event.pull_request.base.sha || github.event.before }}
        run: |
          code=true
          if [ -n "$BASE" ] && [ "$BASE" != 0000000000000000000000000000000000000000 ] \
             && git fetch -q --depth=1 origin "$BASE" \
             && ! git diff --name-only "$BASE" HEAD \
                  | grep -qvE '^(docs|plans|inbox)/|^\.agents/memory/|^\.claude/|^\.codex/|^[^/]+\.md$'; then
            code=false
          fi
          echo "code=$code" >> "$GITHUB_OUTPUT"
      # setup steps for the stack, each with its cache, and each with: if: steps.scope.outputs.code == 'true'
      - if: steps.scope.outputs.code == 'true'
        run: bash .agents/check   # exactly how os review runs it
```

A docs-only commit still reports a green `check`, so a required check never hangs. The skip is done inside the job,
never with workflow-level `paths`/`paths-ignore`. Only root-level `*.md` counts as docs, so a site's content
Markdown still builds.

## Local test database (repos with Postgres)

Real Postgres at the production major version on this machine, a clone per test worker, never a remote database:
`.agents/skills/test-tiers/testdb`, configured by `.agents/testdb.conf`. How to set it up, the pytest and vitest
hooks, and the rollback-per-test pattern: [testdb.md](testdb.md).

## Applying it to a repo

1. Measure first: the check's wall time, the `--durations` top 25, and 30 days of CI minutes
   (`gh run list --created ">=<date>" --json workflowName,event,createdAt,updatedAt,conclusion`).
2. Change one thing at a time and measure again. Parallelism (rule 1) and moving billed-service
   tests to T3 (rule 2) usually save the most.
3. A change to `.agents/check` or a workflow goes through `os review` like any code. A
   workflow change that fires on push is still a push that spends compute (`os push-ok`).
