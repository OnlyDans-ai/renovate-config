---
name: website
description: Working on a Vercel-hosted Next.js/Astro marketing site, preview/production
  branches, contact forms, DNS/email deliverability. Use when the repo deploys to Vercel via a
  main-branch webhook, when asked to add or fix a contact form, or when touching DNS/DMARC/SPF
  for a client domain.
---

# Marketing Website

Applies to Danny's marketing-site repos (single-tenant, Next.js or Astro, deploying `main` ->
production via a Vercel webhook). Not the staffcloud app, and not a multi-tenant product.

## Branch model

Two branches only, long-term: `main` (production, auto-deploys on push) and `preview` (the AI
working branch, with a stable preview URL `https://<repo>-git-preview-<scope>.vercel.app/`). The
default answer to "should I create a branch?" is **no: use `preview`.**

Per iteration:

1. `git checkout preview && git pull origin preview` (create from `main` with `--set-upstream`
   if missing).
2. Work and commit directly on `preview`; no feature branches.
3. Push -> Vercel rebuilds the preview URL -> Danny reviews there.
4. On approval: `git checkout main && git merge --ff-only preview && git push origin main`.
5. `preview` stays in place (now equal to `main`). Next iteration starts at 1.

If `preview` diverges non-fast-forward after a merge, reset only when the divergent work already
shipped another way: `git checkout preview && git reset --hard main && git push
--force-with-lease origin preview`.

Avoid: a `feat/<topic>` branch per iteration (fragments Vercel preview URLs and sprawls) ·
working on `main` directly (ships unreviewed) · deleting `preview` after merge · stacked
`feat/*` branches · a long-lived divergent preview; merge approved chunks as you go.

`v0.app` branches (`v0/<repo>-<hash>`) are v0's own deploy targets, outside this workflow; prune
stale ones with `git fetch --prune && git branch -D <stale>`.

## SEO preservation on a rebuild

When the brief is "rebuild it, keep the rankings": URLs, titles, meta descriptions and body copy
that already rank are the asset: capture them verbatim from the live site before touching
anything, and treat that capture as data, not a draft to improve. If the framework has a
`trailingSlash` setting and the indexed URLs carry a trailing slash, match it exactly; flipping it
301s every ranking page. Redirect indexed cruft (`/sample-page`, `/category/uncategorized`, WP
defaults) rather than rebuilding it. Structured data (schema.org) is a factual description, not
ranking-equity content, so it can and should be corrected or upgraded even on a verbatim rebuild.

## Contact / lead forms

Every form handler follows the same shape, whether it emails a contact or a sales inbox:

- **Honeypot field**: an extra hidden input (e.g. `company`) real users never fill. If it has a
  value, return a normal success response and silently discard; never reveal that a bot was
  caught.
- **Control-character stripping**: any field that reaches an email header (name fields
  interpolated into a Subject) must have CR/LF and other control characters stripped before use;
  a raw newline there is a header-injection vector. A multi-line message body only needs the
  nastier control characters removed, not newlines.
- **Length caps and required-field/format validation**: cap every field server-side regardless
  of client-side limits; validate email format before attempting to send.
- **Submit timeout**: bound the outbound send call (10-15s) so a slow upstream provider doesn't
  hang the request.
- **Best-effort rate limiting**: an in-memory per-instance counter blunts bursts but is not a
  real limiter on serverless (instances are ephemeral, not shared). Say so in a comment; escalate
  to Vercel KV or Cloudflare Turnstile only if abuse becomes real.
- **Fail loud on misconfiguration, fail quiet on spam**: if the destination address env var is
  unset, return an error (503) rather than silently dropping leads. If a submission is scored or
  flagged as spam, accept it with a normal 200 so the sender learns nothing, and either discard it
  or deliver it tagged for review.
- **Never leak provider internals**: log the provider's error detail server-side; return a
  generic failure message to the browser.

A corpus-tuned spam scorer (regex signals over the free-text fields, weighted, with reject/flag/
allow thresholds) is worth adding once a form has a real spam sample to tune against; do not
invent signals in advance of evidence.

### Cloudflare Email Sending

The common transport for these sites is the Cloudflare Email Sending REST API
(`POST /accounts/{account_id}/email/sending/send`), not a third-party email API. Requirements:

- The sending domain must be onboarded to Email Sending in the **same Cloudflare account** as the
  token, or sends fail with error `10203` (sending disabled for zone).
- **Onboarding auto-publishes `_dmarc` with `p=reject`.** Before onboarding, check whether the
  domain's existing mail (e.g. Google Workspace) has SPF/DKIM/DMARC already passing. If it
  doesn't, `p=reject` will start bouncing the client's real email; set `_dmarc` to `p=none` in
  the same sitting as onboarding, then tighten later once SPF/DKIM are verified passing.
  Snapshot the existing DMARC record before onboarding and verify it after; don't clobber one
  that already exists.
- API tokens minted from **Manage Account -> Account API Tokens** are account-owned and do not
  validate against `GET /user/tokens/verify` (returns `1000 Invalid API Token` even for a good
  token); verify with `GET /accounts/{account_id}/tokens/verify` instead. These tokens run
  longer than the 40-character user-token format.
- These tokens are account-scoped; Cloudflare has no per-domain option. Restrict by client IP and
  set an expiry where the dashboard allows it.
