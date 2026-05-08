# Features

Plans for cross-cutting features — anything whose implementation spans
more than one sub-repo and benefits from a shared spec.

A feature plan is **the orchestrator's contribution** to a piece of
work that will be executed in the sub-kits. The plan answers:

- What is this feature?
- Why now?
- What contract changes does it require?
- What does each sub-repo need to do?
- What does "done" look like, observable from outside?

Sub-kits read the relevant `features/<id>.md` before starting work.

## Naming

`features/YYYY-MM-DD-short-name.md`

## When to write one

- Feature touches 2+ sub-repos
- Has non-trivial coordination (ordering, contract changes, migration)
- Will likely span multiple sessions / weeks

## When NOT to

- Single-repo work — just file a task in that sub-kit
- Internal refactor — use a per-repo plan, not a cross-cutting feature
- Migration that's purely a contract change — that's a migration, not
  a feature (use `migrations/`)

## Feature vs migration

A **migration** is a state transition that must complete; everyone
moves from A to B. Failure to finish leaves the system half-broken.

A **feature** is a new capability; it has a planned shape and an
observable outcome. Failure to finish leaves the system without the
new thing, but otherwise stable.

Most cross-cutting features will *include* one or more migrations.
The feature plan links to them.

## Format

See `_template.md`.
