# Migrations

A **migration** is a coordinated state transition across the stack with
a defined start, blast radius, and close criteria. This is the
operational core of the orchestrator after the audit settles — most
ongoing CTO work is managing migrations.

## Lifecycle

```
opened (in-progress) → validating → closed
```

- **opened** — migration file lives in `active/`. Per-repo state
  tracking begins. Affected sub-repos receive a notice in their
  `.claude/active-migrations.md`.
- **validating** — all per-repo work is merged and deployed; validation
  checklist is being worked.
- **closed** — every per-repo state is `✅` and every validation item
  is checked. File moves to `closed/`. Sub-repo notices are removed.

A migration only closes when **every** affected repo is `✅` and every
validation checkbox is checked. No partial closes.

## When to open one

Open a migration when a change has **cross-repo blast radius**:

- Database schema changes that affect API + clients
- Auth model changes (token format, scopes, claims)
- Contract changes (endpoint added/changed/removed, model field
  added/changed/removed)
- Event schema changes
- Infra changes that change runtime behavior visible to apps
- Renaming or reorganizing across repos

Don't open one for:

- Per-repo refactors with no external surface
- Bug fixes that don't change the contract
- Anything that lives entirely inside one sub-repo

## Naming

`migrations/active/YYYY-MM-DD-short-name.md`

Example: `2026-05-07-user-add-tenant-id.md`

## Format

See `_template.md` in this directory for the full shape. Frontmatter is
parseable; body is human-readable.

## Sub-repo notices

When a migration is opened, the orchestrator writes
`.claude/active-migrations.md` into each affected sub-repo. The file
lists open migration IDs touching that repo with one-line summaries and
a link back to the migration file. Sub-kits read this on session start.

When a migration closes, the entry is removed from each sub-repo's
notice file. If no migrations remain, the file is deleted.

This is the **only** path the orchestrator writes into sub-repos.

## Detecting drift

The orchestrator surfaces a migration as **stale** when its per-repo
state hasn't moved in 14+ days while at least one repo is still
unfinished. Stale migrations are the signal that "we started something
and never finished" — exactly the failure mode this whole structure
prevents.

## Don't

- **Don't auto-close.** Closure is a deliberate user-confirmed action.
- **Don't merge migrations.** If two changes are coupled, that's one
  migration, not two; if they're independent, keep them separate.
- **Don't skip the contract update.** A migration that changes
  contracts must update `contracts/` files in the same commit (or
  link to the commit that did).
