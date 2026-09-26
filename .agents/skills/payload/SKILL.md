---
name: payload
description: Payload CMS 3.x gotchas - access control the Local API skips, hook transactions and loops, fields,
  types, admin components and plugins. Use when working in a repo with payload.config.ts, and when debugging access,
  validation, relationship queries, transactions or hook behavior in Payload.
---

# Payload CMS

These are the Payload 3.x behaviors that most often cause bugs. For anything else, read the current docs at
payloadcms.com/docs; the API changes between majors.

## Access control

**The Local API skips access control by default, even when you pass `user`.** An operation done for a user needs
`overrideAccess: false`; keep the default `true` for trusted server work (cron, system tasks) only.

```ts
await payload.find({ collection: 'posts', user, overrideAccess: false })
```

Collection and global access functions may return a query (row-level: `{ author: { equals: user.id } }`);
field-level access returns a boolean only.

## Hooks

- A nested operation inside a hook passes `req` (`req.payload.create({ ..., req })`), or it runs in its own
  transaction and can commit while the outer operation fails.
- A hook that triggers its own operation loops. Pass `context: { skipHooks: true }` on the nested call and return
  early when the hook sees it.

## Fields and queries

- A computed field is an existing type with `virtual: true`; there is no `type: 'virtual'`.
- Relationship `depth` defaults to 2; set `depth: 0` for IDs only, and `select` to limit fields.
- Enabling `versions.drafts` adds a `_status` field.
- `point` fields are not supported on SQLite. MongoDB transactions need a replica set; SQLite transactions are off
  by default.

## Types, admin and plugins

- Run `generate:types` after every schema change; a stale `payload-types.ts` compiles against fields that no longer
  exist. SQL adapters change the schema through Payload's migrations.
- A new custom admin component (`admin.components`) needs the import map regenerated, not just the file.
- A plugin is `(options) => (config) => config`. When it adds fields or hooks to an existing collection, it spreads
  the existing arrays; replacing them silently deletes what the app defined.
