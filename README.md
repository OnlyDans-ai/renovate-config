# renovate-config

OnlyDans org-wide Renovate policy — the ONE home for dependency-update rules (hub-and-spoke, per Ironworks doctrine).

Every repo consumes it with a 3-line `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>OnlyDans-ai/renovate-config"]
}
```

Policy summary: weekly Monday batch (America/Chicago) · security PRs fire immediately, any day · lockfiles regenerate WITH every bump · dev-deps + GitHub Actions patch/minor automerge on green CI · **majors always wait for Danny** (Dependency Dashboard checkbox) · Dependabot ALERTS stay on everywhere, Dependabot version PRs stay off (one bot, one policy).

Repo-specific rules (e.g. staffcloud's Cloud Run image manager, Python 3.13 pins) live in that repo's `renovate.json` AFTER the extends line.
