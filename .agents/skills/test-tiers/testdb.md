# Local test database (testdb)

Real Postgres at the production major version, never SQLite or mocks. A template per migration state, a clone per
test worker, a rolled-back transaction per test, all on a RAM-backed server that runs only on this machine.
`.agents/skills/test-tiers/testdb` runs it from `.agents/testdb.conf`; the design, and what stays on Neon (the migration
rehearsal and the nightly schema-drift check), is in `~/projects/os/plans/test-db-2026-09-25.md`.

```sh
# .agents/testdb.conf (read, never sourced; its own content keys the template too)
TESTDB_IMAGE=pgvector/pgvector:pg17@sha256:…      # production's major version, pinned
TESTDB_MIGRATE='uv run alembic upgrade head'       # runs with DATABASE_URL set to the template being built
TESTDB_HASH='alembic/ alembic.ini tests/seed/'     # every file the template is built from, the seed's code included
TESTDB_ROLES=tests/db/roles.sql                    # optional, idempotent; run as superuser before migrate
TESTDB_SEED='uv run python -m tests.seed'          # optional: synthetic baseline rows baked into the template
TESTDB_ENV='DATABASE_URI=postgres DATABASE_URI_APP=app'   # optional: `testdb env` prints each VAR as that role
```

- **One server per image and roles, a template per migration state.** The template key covers the `TESTDB_HASH`
  files (tracked or untracked), the roles file, this config file and the image. The server key covers the image, the
  roles file's content and `TESTDB_RAM`, so lanes at different migration states share one server and each clones its
  own template; a roles change gets its own server. The name comes from the main checkout, so every worktree shares
  it. The build sets `DATABASE_URL`, `TESTDB_URL` and every `TESTDB_ENV` variable to the template being built, as
  superuser, so migrations that read `DATABASE_URI` work unchanged. For a migrate or seed step that must run as one
  of the app's roles (tables owned by the migrate role, as in production), it also sets `TESTDB_URL_<ROLE>` for each
  role `TESTDB_ENV` names (`TESTDB_URL_APP_MIGRATE` for `app_migrate`).
- **Migrate and seed leave roles alone.** Roles are global to a server, so a role change in a migration would reach
  every state on it. The build hashes each role's `pg_authid` row, memberships and cluster-wide settings inside
  Postgres before and after migrate and seed; a difference fails the build naming the roles, and the server leaves
  service under a new name (`<name>-x<epoch>`): clones already handed out keep working, the next `url` starts a clean
  server, and reap removes the old one once its owners are gone. Role changes go in `TESTDB_ROLES`; a repo whose
  migrations must change roles sets `TESTDB_SERVER=state`, a server per migration state, with no guard. A test that
  changes roles reaches every lane on the server the same way, and trips the guard if a build runs meanwhile.
- **Templates are reaped too.** Each is marked when used. One idle an hour is dropped, never the caller's current
  one; a server holds at most four, the least recently used going first before a new one is built, but never one
  used in the last ten minutes (the cap is exceeded instead, and said). A clone whose template was dropped a moment
  before rebuilds it and clones once more.
- **The roles file and this config are never git-ignored.** Both key the template and CI must have them; testdb
  refuses an ignored one by name (a `*.sql` rule once hid `tests/db/roles.sql`).
- **`testdb build`** builds the template, or waits for a build already running, and makes no clone. A gate calls it
  before its parallel lane so no suite's timing carries the build. Builds on one server run one at a time. A build's
  output goes to a log in the git dir (`testdb-<server>-<template>.build.log`); a failed build prints its tail and
  path.
- **Each clone is leased to the process it was made for**: the test runner's worker, found above any `sh -c` in
  between (or `TESTDB_OWNER=<pid>` from a shell that allocates for a longer-lived process). The next `url` drops a
  clone whose owner has exited; a clone whose owner cannot be told is dropped after two hours; a live owner's clone
  stays as long as its owner runs.
  A server idle a day with no live owner is removed. Nothing restarts a server after a reboot; the next `url` does.
- **The guard runs first.** `url`, `env` and `up` exit 2 before anything connects when any variable in the
  environment, whatever its name, holds a Postgres URL that is not loopback (a `host`, `hostaddr`, `service` or
  `dbname` query key, encoded or not, counts as not loopback), a key=value connection string with a remote host, or
  `PGHOST`/`PGSERVICE` pointing elsewhere. `testdb guard` is the check alone. A variable tests never read goes in
  `TESTDB_IGNORE_ENV` in the repo's file, never in the environment. Run it after the app's own settings load (a
  dotenv file read later would bypass it), so the pytest hook below imports settings first where the repo has them.
- **Test as the app's role.** `testdb url --role app` connects as that role, so row-level security applies. testdb
  refuses a role that is SUPERUSER, BYPASSRLS, cannot log in, or owns an RLS table without `FORCE ROW LEVEL SECURITY`,
  directly or through a role it inherits.
  A repo with RLS keeps one cross-tenant test that reads through that URL and expects no rows.
- **Needs** docker, git and bash; psql runs inside the container. CI runs the same way (GitHub's Ubuntu runners have
  docker): no service container.

pytest, one clone per process (the xdist controller and each worker), allocated before collection so module-level
imports already see it (conftest.py at the repo top):

```python
import os, subprocess
TESTDB = os.path.join(os.path.dirname(__file__), ".agents", "skills", "test-tiers", "testdb")

def pytest_configure(config):
    # import the app's settings here first if it loads a dotenv file, so the guard sees what the tests will
    config._testdb_url = subprocess.check_output(["bash", TESTDB, "url"], text=True).strip()   # guard, then clone
    os.environ["DATABASE_URL"] = config._testdb_url

def pytest_unconfigure(config):
    if getattr(config, "_testdb_url", None):
        subprocess.run(["bash", TESTDB, "drop", config._testdb_url], check=False)
```

vitest runs a setup file once per test file, so this gives each test file its own clone: about 0.13 s, spread
across the workers, and no file sees another's rows even when a worker runs many files:

```ts
// vitest.config.ts: test.setupFiles: ["tests/testdb.setup.ts"]
import { execFileSync } from "node:child_process";
import { afterAll } from "vitest";
const TESTDB = ".agents/skills/test-tiers/testdb";
const urls = execFileSync("bash", [TESTDB, "env"], { encoding: "utf8" }).trim().split("\n");   // guard first; needs TESTDB_ENV
for (const line of urls) { const at = line.indexOf("="); process.env[line.slice(0, at)] = line.slice(at + 1); }
afterAll(() => execFileSync("bash", [TESTDB, "drop", urls[0].slice(urls[0].indexOf("=") + 1)]));
```

Bun: a `--preload` on the command line (in `bunfig.toml` it would hit every unit test too), one clone per `bun test`
process. A thrown error in a preload reads as one failed test, so a refusal exits the process instead; and
`bun --env-file=.env.test` pins the one dotenv file a run loads, so the guard sees what the tests will:

```ts
// bun --env-file=.env.test test --preload ./tests/testdb.preload.ts <file>
import { afterAll } from "bun:test";
import { execFileSync, spawnSync } from "node:child_process";
const TESTDB = new URL("../.agents/skills/test-tiers/testdb", import.meta.url).pathname;
const got = spawnSync("bash", [TESTDB, "env"], { encoding: "utf8", stdio: ["ignore", "pipe", "inherit"] });
if (got.status !== 0) process.exit(got.status || 1);   // testdb said why on stderr; no test ran
const lines = got.stdout.trim().split("\n");
for (const line of lines) { const at = line.indexOf("="); process.env[line.slice(0, at)] = line.slice(at + 1); }
afterAll(() => execFileSync("bash", [TESTDB, "drop", lines[0].slice(lines[0].indexOf("=") + 1)]));
```

**One transaction per test, rolled back.** Tests that share a clone each run inside a transaction the fixture rolls
back: in pytest, a function fixture that opens a connection, begins, binds the app's session to it and rolls back
after the test (SQLAlchemy: `session = Session(bind=conn, join_transaction_mode="create_savepoint")`); in vitest, a
`beforeEach`/`afterEach` pair around the repo's client (`BEGIN` / `ROLLBACK`). Each repo proves it with one test pair:
the first writes a row and fails, the second finds no such row. A test that must commit across connections gets its
own `testdb url`.
