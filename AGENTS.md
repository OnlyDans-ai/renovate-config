# renovate-config

<!-- os:core:start -->
# Working agreement

Danny owns goals, priorities, spend, and which harness drives. You own delivery: complete the
authorized task with its checks passing, fix defects your change causes, and report unrelated
findings instead of expanding scope. Recommend the best option with the deciding reason. Ask only
for what you cannot resolve yourself: credentials, spend, priority, or a scope change.

## Boundaries

- Secrets never enter chat, a tracked file, or a command's output. Provisioning goes through
  1Password (`op-sa.sh <account> read`) or a hidden-input script Danny runs himself.
- A push that fires a deploy or spends metered compute needs Danny's word (`os push-ok <hours>`).
  Production promotes are asked separately, every time.
- Destructive or irreversible actions need explicit confirmation: force-push, history rewrite,
  data deletion, killing another project's processes, sending anything to a person.
- Anything handed to Danny to run is one short line or `bash <script>`.
- Todoist tasks only when explicitly asked. `gh` calls carry `--json` or `--jq`; local git first.

## Review scales with risk, by judgment

- Cosmetic, docs, tests, tooling: self-check, checks green.
- Ordinary changes: the outside voices review the code (`os review`, tier 2), each from its own
  perspective.
- Auth, money, migrations, security, destructive data, cross-tenant: another family reviews the
  plan first, then both other families review the code.
- A review voice that is out never stops work: the unit closes on the voices that answer (none
  answering still stops it), and `os review --sweep` has the absent voice read it when it is back.
- Deterministic checks run before any model, and a review unit is at most 400 changed lines:
  `os review` refuses a larger diff and prints the split. Tier 2 ships a risk map, tier 3 a
  boundary table (what the change reads, writes, spawns, parses, and how each path fails);
  reviewers read against it. Two model passes per unit: one finds, one verifies the fix delta
  against its tests. What the second still finds is fixed in one batch and closed by a test or
  by Danny's word, never by a third pass and never by silence. Two findings on one check mean
  redesign that check, not patch it. Read a diff whole and form your verdict before reading
  anyone else's. Verify claims yourself; a report is not evidence.

## Working state lives in the repo

- Run `os boot` first (it fixes this repo's OS drift and is silent otherwise). Then read
  `docs/HANDOFF.md`, then the two memory indexes: `.agents/memory/MEMORY.md` (this repo) and
  `~/.agents/memory/MEMORY.md` (what holds across repos). Write the handoff at close. Plans live
  in `plans/`. A durable fact becomes a topic file in the memory it belongs to, indexed by one line:
  what is specific to this repo in `.agents/memory/`; research on a tool, vendor, service or model
  that other repos could use in `~/.agents/memory/<topic>.md`, one file per topic, edited in place
  by any repo, and linked from this repo's memory rather than copied. Before re-deriving a gotcha
  or a decision, search what is known: `os recall <words>`. Chat history does not travel between
  harnesses.
- The project section of `AGENTS.md` (below `os:project:start`) is this repo's own and reads the
  same on every harness: how to run, test and deploy it and the boundaries every turn needs, a
  page (60 lines) at most; a standing fact goes to memory, state to the handoff, and a rule only
  one harness can follow goes in that harness's own folder in the repo (`.claude/`, `.codex/`),
  pointed to from the section. The core block above it is the OS's; `os sync` rewrites it.
- An OS defect or a suggestion for the OS (the `os` CLI, a guard, a skill, a rendered file, or a
  harness binding the OS deploys: a deny rule, a hook, a sandbox root, a setting, on any harness
  including the one you run on), met in any repo: work around it locally, then file a case at
  `~/projects/os/inbox/<date>-<repo>-<slug>.md` (shape in that folder's README); if that path
  cannot be written from your sandbox, park the case under `## OS case` at the top of
  `docs/HANDOFF.md` and say so at close. Never edit the OS or a harness binding from a project
  session. The OS session triages the inbox and the parked cases at its start: adopt, absorb,
  route to the harness that owns the binding, or reject with the reason kept in `inbox/rejected/`.
- Delegate big reads and disjoint lanes to agents; keep judgment here. Delegate at the
  subagent model your harness's slot in the DESIGN § 1 matrix names (volume is low, so it is
  chosen for capability, not price); security work runs at the strongest model the harness has.
  A loop needs an independent checker and a stop condition; the maker never grades its own work.

## Same rules on every harness

This file is the contract, and it reads the same from Claude Code, Codex, Gemini, OpenCode or any
other harness. Prefer the agnostic form of anything: repo files, git hooks, plain scripts. Where a
harness needs its own binding (an instruction pointer, a hook, a deny rule, a setting), that binding
carries these rules unchanged; it never adds, drops or softens one. Skills are files,
`~/.os/skills/<name>/SKILL.md`, copied into each repo's `.agents/skills/`: use one by reading its
file or running the script it names; a harness slash command is a convenience, never the only door.

Each harness owns its own binding and nothing else: its config directory, its settings (model,
effort, compaction, sandbox), its hooks and deny rules, and its column in the DESIGN § 1 matrix.
It keeps a ledger at `~/.os/harness/<name>.md`: how each OS default is bound there, and what it
cannot do, dated, with evidence. A gap is logged, never papered over; `os check <name>` reads it.
The core and another harness's binding are proposed to, never edited. Onboarding a harness is:
connect it, hand it `~/.os/harness/README.md`, and it binds itself.
<!-- os:core:end -->

<!-- os:project:start -->
## This project

Stack: one Renovate preset, `default.json`, consumed by every OnlyDans repo as `github>OnlyDans-ai/renovate-config`
Run: nothing runs here; Renovate reads `default.json` from `main`
Test: `bash .agents/check` (`default.json` parses and keeps its required keys; under 1 s, offline)

Deploy branches: see `.agents/deploy-branches`. A push to one of them needs `os push-ok` first.

<!-- Project-specific boundaries, conventions, and exceptions go below this line. `os sync`
     never touches anything below `os:project:start`. -->

A push to `main` is a production change for every repo that extends this preset, at the next
Renovate run. Treat an edit to `default.json` like a deploy: `os push-ok` first, and say in the
commit which repos the rule is meant for. README.md is the policy summary; keep it true to the
JSON in the same commit. Repo-specific rules belong in that repo's own `renovate.json`, after the
extends line, never here.
<!-- os:project:end -->
