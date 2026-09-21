---
name: payload
description: Working with Payload CMS (payload.config.ts, collections, fields, hooks, access
  control, Local/REST/GraphQL API). Use when debugging validation errors, security issues,
  relationship queries, transactions, or hook behaviour in a Payload project.
---

# Payload CMS

Payload is a Next.js-native, TypeScript-first CMS: admin panel, database layer, REST/GraphQL
APIs, auth, and file storage, all driven from one config. This covers Payload 3.x. For anything
not here, check current docs (payloadcms.com/docs) rather than trusting a stale copy; the API
has moved fast across majors.

## Minimal config shape

```ts
import { buildConfig } from 'payload'
export default buildConfig({
  admin: { user: 'users' },
  collections: [Users, Media, Posts],
  editor: lexicalEditor(),
  secret: process.env.PAYLOAD_SECRET,
  db: mongooseAdapter({ url: process.env.DATABASE_URL }), // or postgresAdapter / sqliteAdapter
})
```

A collection is a slug plus fields plus optional `admin`, `hooks`, `access`, `versions`,
`timestamps`. Globals are the same shape for singleton content (site settings, nav).

## Fields, in brief

Common types: `text`, `richText`, `select`, `relationship`, `upload`, `array`, `blocks`, `point`,
and `join` (reverse relationship). A computed field is an existing type with `virtual: true`
(there is no `type: 'virtual'`), populated by an `afterRead` hook or a relationship path. Fields
support `admin.condition` for conditional display and a `validate` function for custom rules.
`filterOptions` on a `relationship` field restricts which documents can be selected.

## Access control

```ts
export const ownPostsOnly: Access = ({ req }) => {
  const user = req.user as User
  if (!user) return false
  if (user.roles?.includes('admin')) return true
  return { author: { equals: user.id } } // row-level, not just boolean
}
```

Collection- and global-level access functions can return a query constraint (row-level security);
field-level access can only return a boolean, never a query.

**Local API bypasses access control by default, even when you pass `user`.** This is the most
common security bug in a Payload app:

```ts
// WRONG: user is passed but ignored; every row is returned regardless of permissions
await payload.find({ collection: 'posts', user: someUser })

// RIGHT: overrideAccess: false actually enforces it
await payload.find({ collection: 'posts', user: someUser, overrideAccess: false })
```

Use `overrideAccess: true` (the default) only for trusted server-side work (cron, system tasks);
an operation performed on behalf of a specific user needs `overrideAccess: false`.

## Hooks

`beforeChange`, `afterChange`, `beforeValidate`, `beforeDelete`, etc., at the collection or field
level. Two gotchas:

- **Nested operations need `req` threaded through**, or they run in a separate database
  transaction and can leave data inconsistent if the outer operation later fails:
  `req.payload.create({ ..., req })`.
- **A hook that triggers the same operation type on itself loops.** Guard with a `context` flag:
  check `context.skipHooks` at the top of the hook, and pass `context: { skipHooks: true }` on the
  nested call.

## Queries

Local API (`payload.find`, `payload.findByID`) supports `where` with operators and AND/OR
grouping, `depth` (relationship population depth, default 2; set `0` for IDs only), `select`
(limit returned fields), `sort`, and nested-property filtering. The same query shape works over
REST and GraphQL with minor syntax differences.

## Migrations and types

Run `generate:types` after any schema change: `payload-types.ts` goes stale silently otherwise,
and a stale type will compile against a field that no longer exists. For SQL adapters
(Postgres/SQLite), Payload's migration commands manage schema changes; for MongoDB there's no
migration step but transactions require a replica set.

## Media and uploads

An `upload` field type plus a `Media` collection with `upload: true` handles file storage; a
storage adapter plugin (S3, Cloudflare R2, etc.) redirects where files land in production instead
of local disk.

## Common gotchas

1. Local API bypasses access control unless `overrideAccess: false` is passed with `user`.
2. Missing `req` in a nested hook operation breaks transaction atomicity.
3. Hook loops from an operation re-triggering its own hook; guard with `context`.
4. Field-level access returns a boolean only, never a query constraint.
5. Relationship `depth` defaults to 2; over-fetching is easy to miss until it's slow.
6. Drafts inject a `_status` field automatically once `versions.drafts` is enabled.
7. MongoDB transactions require a replica set; SQLite transactions are off by default.
8. `point` fields (geolocation) are not supported on SQLite.

## Admin customisation

Custom admin components (fields, views, providers) are registered via `admin.components` and
resolved through a generated `importMap`; a new custom component needs a regeneration step, not
just a file.

## Plugins

A plugin is `(options) => (config) => Config`: it receives the existing config and returns a
modified one. When adding fields or hooks to existing collections inside a plugin, always spread
the existing array rather than replacing it, or you silently delete whatever the app already
defined.
