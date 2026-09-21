---
name: frontend-verify
description: Verifying a UI change actually works in the browser, not just that the build and
  tests pass. Use after any frontend component, form, or page change, before calling a UI task
  done, and when debugging a visual or interaction bug.
---

# Frontend Verification

Static checks (build, types, lint, unit tests) prove the code compiles. Only driving the real
rendered page proves it works. Run both, in order, before calling a UI change done.

## Layer order

1. **Build**: the app actually builds for the target environment.
2. **Types**: no type errors introduced.
3. **Lint**: no new lint violations.
4. **Unit tests**: component/logic tests pass.
5. **Real browser**: navigate the actual running app and interact with it.

A change that passes 1-4 but was never opened in a browser is unverified. Do not report a UI
task complete on static checks alone.

## Browser tool: local only, never cloud, for localhost

For any `localhost` / `127.0.0.1` / `*.local` URL, use a **local** browser automation tool
(agent-browser, Claude in Chrome, or a local Playwright run), never a cloud browser tool (cloud
Playwright, Firecrawl, or any hosted browser MCP). A cloud tool runs in remote infrastructure and
physically cannot reach a dev server on this machine; a call that appears to succeed against
`localhost` from a cloud tool is hitting something else, not your app. Cloud tools are correct
for crawling or checking a public production URL; never for a dev server.

Use `localhost:<port>`, not the bare IP form (`127.0.0.1:<port>`); some dev servers block
cross-origin requests to hot-reload endpoints from the IP form, so the page renders a
correct-looking static shell that never hydrates. Every click is then inert while the walk
"passes."

**First action after opening the page: assert hydration.** Interact with one known control and
require an observable change (a DOM update, a URL param, an `aria-pressed` toggle). No observable
change means the page never hydrated; that is a distinct failure from "the feature is broken,"
and a walk over an unhydrated page proves nothing.

## Parallel worktrees: offset ports

When more than one dev server is running (parallel worktrees, multiple branches), each one binds
its own port. Resolve the actual port for the tree you're verifying; a hardcoded `localhost:3000`
silently verifies a different branch's server. If the project has a preflight script or documents
its port convention, use it; otherwise probe and confirm before trusting the result.

## Stale-UI sweep

Before trusting what's on screen, confirm it's actually the code you just wrote:

- Hard reload (bypass cache), not a soft refresh.
- Clear the framework's build cache if the dev server has been running across several changes and
  the UI looks unchanged when it shouldn't.
- Check that the served bundle actually changed: a stale service worker, a cached chunk, or a
  dev server that didn't pick up the file change will show you yesterday's UI while looking fine.

A control that does nothing, or a page showing outdated content, erodes trust faster than a
missing feature. Fix it now rather than logging it as future work.

## States to exercise

For any new or changed component, walk all four states, not just the happy path:

- **Loading**: spinner or skeleton, not a blank flash.
- **Error**: an actionable message, not a raw stack trace or silent failure.
- **Empty**: a message plus a next action, not a bare blank area.
- **Success**: the expected content, matching what the feature is supposed to show.

Also check: console for errors during normal use (fail), network tab for 4xx/5xx on the tested
flow (fail), and consistency with one or two sibling pages if the change touches a shared
component.

## Keyboard and accessibility basics

- Tab through the interactive elements in the changed area; every control must be reachable and
  show a visible focus state.
- Every input has an associated label (visible or `aria-label`), not just a placeholder.
- Buttons and links are real `<button>`/`<a>` elements, not a `<div>` with a click handler.
- An accessibility-tree snapshot (not a screenshot) is the fastest way to catch unlabeled
  controls and incorrect roles: take one after any structural change.

## Evidence

Screenshot or snapshot each state you exercised. "I fixed it, trust me" is not verification:
show what changed and what it looks like now.
