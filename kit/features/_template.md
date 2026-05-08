---
id: YYYY-MM-DD-short-name
opened: YYYY-MM-DD
status: planning           # planning | in-progress | shipped | abandoned
affects: [api, ios, web, devops]
related_migrations: []     # IDs of migrations this feature depends on
related_adrs: []
---

# <Feature name>

## What

<2–4 sentences. The capability being built, described from the outside
— what someone using the system can do after this ships.>

## Why now

<2–3 sentences. The driver. What changed in the world / product / business
that put this on top of stack? If you can't answer this, the feature
isn't ready to plan — move it to roadmap.md "Later".>

## Success criteria

Observable outcomes, not implementation details. A skeptical reviewer
should be able to verify each.

- _(e.g. "iOS user can switch between tenants without logging out")_
- _(e.g. "API rejects cross-tenant access with 403 + audit log entry")_
- _(...)_

## Contract impact

Files in `contracts/` that change. Link to the relevant sections.

- `contracts/models.md` — _(what changes)_
- `contracts/endpoints.md` — _(what changes)_
- `contracts/events.md` — _(what changes, if any)_

## Per-repo plan

### api
- _(scope, key changes, dependencies)_

### ios
- _(scope, key changes, UI considerations)_

### web
- _(...)_

### devops
- _(infra changes, deploy considerations)_

## Migrations included

If this feature requires coordinated state transitions, list the
migration IDs (or note that they'll be opened as work progresses):

- `migrations/active/<id>.md` — _(brief)_

## Sequencing

If sub-repos depend on each other for this feature, state the order
(API ships first → clients consume → devops scales) and the contract
versioning approach for any intermediate state.

## Open questions

- _(things parked for now)_

## Notes

- _(unstructured)_
