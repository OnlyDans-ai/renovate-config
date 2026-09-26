---
name: website
description: Danny's marketing-site conventions - the preview/main branch flow on Vercel, rebuilding a site without
  losing its rankings, contact-form handlers, and Cloudflare Email Sending with DMARC. Use in a single-tenant
  Next.js or Astro site repo that deploys main to Vercel, when adding or fixing a contact form, or when touching DNS,
  DMARC or SPF for a client domain.
---

# Marketing website

For Danny's single-tenant marketing sites (Next.js or Astro, `main` deploying to production through Vercel). Not
staffcloud and not a multi-tenant product.

## Branches

Two long-lived branches: `main` (production) and `preview` (where agents work, with a stable preview URL
`https://<repo>-git-preview-<scope>.vercel.app/`). The answer to "should I make a branch?" is no: work on `preview`.

1. `git checkout preview && git pull origin preview` (create it from `main` if missing).
2. Commit on `preview` and push; Danny reviews on the preview URL. Both pushes deploy, so each needs his push grant.
3. On his approval: `git checkout main && git merge --ff-only preview && git push origin main`. `preview` stays.

If `preview` stops fast-forwarding after a merge and its extra work already shipped another way, resetting it to
`main` is a force-push: ask Danny first. `v0/<repo>-<hash>` branches belong to v0.app, outside this flow; deleting
stale ones is Danny's (hand him the `git branch -D` line).

## Rebuilding without losing rankings

The URLs, titles, meta descriptions and body copy that rank are the asset. Capture them verbatim from the live site
before touching anything and treat the capture as data, not a draft. Match the indexed trailing-slash form exactly;
flipping `trailingSlash` 301s every ranking page. Redirect indexed WordPress cruft (`/sample-page`,
`/category/uncategorized`) instead of rebuilding it. Structured data is a factual description, so correct and upgrade
it freely.

## Contact forms

Beyond the usual honeypot, header-injection stripping, server-side length caps and a 10-15 s send timeout:

- An in-memory rate limit only blunts bursts on serverless; say so in a comment, and move to Vercel KV or Turnstile
  only when abuse is real.
- Fail loud on misconfiguration (destination env var unset → 503, never dropped leads); fail quiet on spam (a caught
  bot gets a normal 200). Log provider errors server-side; the browser gets a generic message.
- Add a spam scorer only once there is a real spam sample to tune it on.

## Cloudflare Email Sending

The usual transport is `POST /accounts/{account_id}/email/sending/send`.

- The sending domain must be onboarded in the same Cloudflare account as the token, or sends fail with `10203`.
- **Onboarding publishes `_dmarc` with `p=reject`.** Snapshot the existing DMARC record first. If the domain's real
  mail (e.g. Google Workspace) does not already pass SPF and DKIM, set `p=none` in the same sitting and tighten later;
  never clobber an existing record.
- Account API tokens fail `GET /user/tokens/verify` with `1000 Invalid API Token`; verify them with
  `GET /accounts/{account_id}/tokens/verify`. They are account-scoped (no per-domain option): restrict by IP and set
  an expiry.
