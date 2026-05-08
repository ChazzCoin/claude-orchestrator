---
id: YYYY-MM-DD-short-name
opened: YYYY-MM-DD
status: in-progress       # in-progress | validating | closed
affects: [api, ios, web, devops]   # list of sub-repo names from stack/inventory.md
contracts_changed: []     # e.g. [models.User, endpoints.users, events.user_created]
related_adrs: []          # e.g. [2026-04-30-multi-tenant]
---

# <Short title>

## What changed

<2–4 sentences. The actual change in technical terms — schema, contract,
behavior. Be specific. "Adding tenant_id (UUID, NOT NULL) to users table"
not "multi-tenancy work.">

## Why

<2–4 sentences. The driver — product requirement, compliance, scale,
debt. Future-you reading this in a year should understand the motivation
without reconstructing context.>

## Per-repo state

- **api** — _(not started | in progress: PR #N | merged | deployed | ✅)_
- **ios** — _(...)_
- **web** — _(...)_
- **devops** — _(...)_

Use ⚪ not started · 🟡 in progress · ✅ done. Update as work moves.

## Plan

Per-repo summary of what each kit needs to do. This is the spec each
sub-kit reads when picking up the work.

### api
- _(what changes in the API repo)_

### ios
- _(what changes in iOS)_

### web
- _(...)_

### devops
- _(infra change, migration script, deploy ordering)_

## Validation

Cross-cutting checks that prove the migration actually works end-to-end.
A migration cannot close until every box is checked.

- [ ] _(specific behavior visible from one platform proving another did its part)_
- [ ] _(data integrity check — backfill complete, no orphans)_
- [ ] _(deploy ordering verified — no broken intermediate states)_
- [ ] _(rollback path tested or accepted)_

## Sequence / dependencies

If order matters (e.g. API ships before clients can deploy), document
it here. If everything can ship independently, say so explicitly.

- _(populate)_

## Rollback

What does rolling this back look like? If it's not reversible (e.g.
once data is backfilled), say so explicitly and document the
forward-fix path instead.

- _(populate)_

## Notes

Anything unstructured worth keeping — surprises, decisions made mid-flight,
links to PRs, links to incidents, links to ADRs.

- _(populate)_
