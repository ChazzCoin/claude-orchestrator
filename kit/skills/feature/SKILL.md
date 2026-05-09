---
name: feature
description: Author a cross-cutting feature plan — what's being built, why now, contract impact, per-repo plan, success criteria. Stored in features/YYYY-MM-DD-short-name.md. Triggered by "/feature", "plan a feature", "cross-cutting feature", "we want to build X across the stack".
---

# /feature — Cross-cutting feature plan

Author a feature plan in `features/` for work that spans more than one
sub-repo. The feature plan is the orchestrator's contribution to a
piece of work that sub-kits will execute.

**Output pattern:** the rendered feature plan follows
`features/_template.md` markdown structure (durable artifact, not
catalogued). Chat output during drafting uses
[Pattern 23 — Activity timeline](../../output-catalogue.md#23--activity-timeline)
for the per-repo plan walkthrough and
[Pattern 1 — Hero completion card](../../output-catalogue.md#1--hero-completion-card)
when the file lands. For impact assessment, may compose with
[Pattern 4 — Sprint task board](../../output-catalogue.md#4--sprint-task-board)
showing per-repo work.

## When this is the right shape

- Feature touches 2+ sub-repos
- Has non-trivial coordination (ordering, contract changes, migration)
- Will likely span multiple sessions or weeks

## When NOT

- Single-repo work — file a task in the sub-kit
- Internal refactor — use a per-repo plan
- Migration without new capability — use `/migration` directly

## Feature vs migration

A **migration** is a state transition that must complete; everyone
moves from A to B.

A **feature** is a new capability with planned shape and observable
outcome.

Most cross-cutting features include one or more migrations. The
feature plan links to them.

## Behavior contract

- **Pull from the user, not from training.** The feature plan is
  specific to this stack. Read `stack/inventory.md`,
  `platform-constraints.md`, and relevant `contracts/` files for
  context, then ask the user the questions specific to this feature.
- **Define success outside-in.** "What does someone using the system
  see when this works?" — not "what do we implement."
- **Surface contract impact early.** Most cross-cutting features
  change contracts; identify the affected files in
  `contracts/` first.

## Process

### Step 1 — Surface the feature

Ask, one or two at a time:

1. **What's the capability?** One paragraph from outside the system.
2. **Why now?** What's the driver? If it can't answer this, move it
   to `roadmap.md` Later instead.
3. **Which sub-repos are involved?** And which is most affected?
4. **What contract changes are required?** (Models, endpoints, events)
5. **What does success look like?** Observable, verifiable outcomes.
6. **Any sequencing constraints?** Does anything need to ship before
   anything else?

### Step 2 — Generate ID and draft

ID: `YYYY-MM-DD-short-name`. File: `features/YYYY-MM-DD-short-name.md`,
using `features/_template.md` as base.

### Step 3 — Identify migrations

If the feature requires coordinated state transitions, identify them.
Don't open them yet — note them in the feature plan's "Migrations
included" section. The user opens them via `/migration` when ready.

### Step 4 — Show, confirm, write

Show the feature draft. User confirms. Write. Don't auto-commit.

## Style rules

- **Outside-in success criteria.** Not "implement endpoint X" — "user
  can switch tenants without re-login."
- **Per-repo plan is a sketch, not a spec.** Each sub-kit will produce
  the detailed implementation plan for its repo. The feature plan is
  the shared context they all reference.
- **Don't pad with implementation details.** The feature file should
  fit in 2 screens.

## What you must NOT do

- **Don't open migrations** during feature drafting. List them, let
  the user decide when to open via `/migration`.
- **Don't write per-repo task files** into sub-repos. The feature plan
  is the orchestrator-side artifact; sub-kits author their own tasks.
- **Don't auto-commit.**

## When NOT to use this skill

- **Migration without new capability** — `/migration`
- **Single-repo feature** — file in sub-kit
- **Architectural decision** — `/decision`

## What "done" looks like

Feature plan drafted at `features/YYYY-MM-DD-short-name.md`. Any
contract updates drafted alongside if obvious. Migration IDs listed
under "Migrations included" but not opened. Shown to user,
uncommitted.
