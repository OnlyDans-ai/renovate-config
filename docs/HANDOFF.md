# Handoff — renovate-config

Updated 2026-09-21 18:00 CDT (onboarded onto the OS; the preset itself is unchanged).

## State

- **The preset.** `default.json` is the OnlyDans org-wide Renovate policy: weekly Monday batch
  (America/Chicago), security PRs at once, lockfile regeneration with every bump, automerge for
  dev-dependency and GitHub Actions patch/minor on green CI, majors wait for Danny. Every repo
  extends it as `github>OnlyDans-ai/renovate-config`. Last policy change: `332e276`, 2026-07-26.
- **On the OS since 2026-09-21.** Core block and project section in AGENTS.md; CLAUDE.md is the
  `@AGENTS.md` line; `.githooks/pre-commit` (secret scan) and `.githooks/pre-push`
  (deploy-branch grant). `.agents/check` is gate 0: `default.json` parses and keeps its
  required keys (offline, under 1 s). `.agents/deploy-branches`: `main`, because a push there
  changes dependency policy for every consuming repo at Renovate's next run.
- The GitHub repo is **public**. The onboarding commit is local only.

## Next action

- Nothing queued. Before any policy edit, run Renovate's own validator by hand
  (`npx --package renovate renovate-config-validator default.json`; it downloads all of Renovate).

## Open decisions (Danny)

- If the GitHub repo is public (Renovate presets often must be readable by the app), pushing
  this commit publishes the OS scaffold: AGENTS.md's working agreement, the skills, the MCP
  file. Nothing in it is a secret, but it is Danny's call whether it goes out.

## Constraints

- Repo-specific rules live in that repo's `renovate.json`, never here.
- README.md and `default.json` change together.
