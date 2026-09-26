---
name: frontend-verify
description: Checks a UI change in a real browser against the running dev server, with the local-browser, port and
  hydration gotchas that make a browser check pass falsely. Use after changing a frontend component, form or page,
  before calling a UI task done, and when debugging a visual or interaction bug.
---

# Frontend verification

A UI change is verified only when the running page has been driven in a browser; build, types and tests passing do
not show that it works.

- **Local browser for local servers.** For `localhost`, `127.0.0.1` or `*.local`, use a browser on this machine
  (Claude in Chrome, agent-browser, a local Playwright run). A cloud browser tool cannot reach this machine, so a call
  that seems to succeed against `localhost` from one is hitting something else. Cloud tools are for public URLs.
- **`localhost:<port>`, not `127.0.0.1:<port>`.** Some dev servers refuse hot-reload requests from the IP form, and
  the page renders a static shell that never hydrates.
- **Prove hydration first.** Operate one known control and require a visible change (DOM, URL, `aria-pressed`).
  No change means the page never hydrated, which is a different failure from a broken feature.
- **The right port.** Parallel worktrees each run their own server; find the port for the tree under test (the
  project's preflight script or port convention, else probe) instead of assuming `:3000`.
- **The right build.** Hard-reload; if the UI has not changed when it should, suspect a stale service worker, a
  cached chunk or a dev server that missed the file change before suspecting the code.

Walk the loading, error, empty and success states of what changed, with the console and network panel open
(errors and 4xx/5xx on the flow fail it). Take an accessibility-tree snapshot after structural changes; it shows
unlabeled controls and wrong roles faster than a screenshot. Report with a screenshot or snapshot of each state.
