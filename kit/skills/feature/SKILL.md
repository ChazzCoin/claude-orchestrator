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

---

## Modes

This skill has two modes. Detect from user input:

- **Open** — "/feature", "plan a feature", "we want to build X
  across the stack" → author a new feature plan and drop notices
  into each affected sub-repo.
- **Refresh** — "/feature refresh", "re-sync feature notices",
  "feature status changed, update notices" → re-render
  `<sub>/.claude/active-features.md` for affected sub-repos without
  authoring a new feature. Useful after editing a feature's
  `status:` frontmatter or its `affects:` list by hand.

If ambiguous, ask.

---

## Notice file format

Per the auto-managed orchestrator → sub-repo discipline (see
[`kit/templates/sub-repo-notices/README.md`](../../templates/sub-repo-notices/README.md)),
this skill owns `<sub>/.claude/active-features.md`. When writing it:

- **Use the template at
  `.claude/templates/sub-repo-notices/features.md.template`** in
  this orchestrator instance. It defines the header (auto-managed
  warning + source-orchestrator pointer + timestamp), the body
  shape, and the comment hint inside `## Open` describing each
  entry's structure.
- **Substitute placeholders** when rendering:
  `{{ORCHESTRATOR_PATH}}` — absolute path to this orchestrator
  instance; `{{REPO_NAME}}` — the sub-repo name from
  `state/manifest.md`; `{{TIMESTAMP}}` — ISO date+time at write
  time; `{{SKILL}}` — `/feature`;
  `{{FEATURE_ENTRIES}}` — the rendered entry list.
- **Regenerate the body wholesale** every time. Don't read the
  existing file and merge — that's how stale entries accumulate.
  Read `features/*.md`, filter for entries whose frontmatter
  `affects:` contains this repo AND `status:` is `planning` or
  `in-progress`, render in directory order (newest first by ID date
  prefix).
- **Delete the file when the entry list is empty.** No empty notice
  files left behind.
- **Don't invent new sections.** If the format needs to change,
  update the template in the kit, then `/sync` instances.

This sits alongside `/migration`'s `active-migrations.md` ownership;
each concern owns its own file. Don't combine into a single
multi-section notice file.

---

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
- **Write per-repo notices only after user confirmation.** Each
  affected sub-repo gets `.claude/active-features.md` regenerated.
  This is a write into another git repo — show the rendered output
  before writing.
- **Skip notice writes for sub-repos without a local clone.** If
  `repos/<name>/` doesn't exist, surface that to the user; offer to
  run `bin/setup` or skip the notice for that repo. Don't fail the
  whole flow over one missing clone.
- **Preferences-aware.** The notice-write confirmation in Step 5
  of Mode: Open is registered as
  `auto-write-active-features-notices` (low-risk). Follows the
  recipe at [`.claude/preferences.md`](../../preferences.md)
  "Skill recipe at a known fork": read `state/preferences.md`
  first; if found, write notices without asking (disclose on
  first apply per session); otherwise ask normally, log to
  `state/decision-log.md`, run streak-threshold offer.

## Mode: Open

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

### Step 4 — Draft sub-repo notices

For each repo in the feature's `affects:` list:

- Path: `repos/<name>/.claude/active-features.md`
- The sub-repo's working tree is at `repos/<name>/` (convention).
  Notice writes require a local clone. If `repos/<name>/` doesn't
  exist, surface in step 5 — user can run `bin/setup` to clone or
  skip the notice for that sub-repo.
- Render the file content **wholesale** from `features/*.md` using
  the template per "Notice file format" above. The new feature is
  now drafted into `features/`, so it lands in the rendered output
  naturally; older still-open features affecting this repo also
  stay in.
- Don't write yet — show the rendered content as part of step 5.

If a feature is being authored that does NOT yet exist in
`features/` on disk (because it's drafted in step 2 but not yet
written), include it in the rendered output **as if it were
already written** — the notice should reflect what the post-write
state will be.

### Step 5 — Show everything, confirm

Show together:

- The drafted feature file
- Any contract updates drafted alongside (if obvious)
- The per-repo `active-features.md` files (one per affected
  sub-repo, or a "skipped — clone missing" line where applicable)
- The migration IDs listed under "Migrations included" with a
  reminder that they're not yet opened

User confirms before writing.

### Step 6 — Write

Write all files: the feature plan, contract updates, and the
per-repo notices. Don't auto-commit — the user reviews and commits.

For each per-repo notice: if the rendered entry list would be
empty, **delete** any existing `active-features.md` rather than
writing an empty one. (This case arises mostly during the Refresh
mode below, but applies in Open if the new feature was added then
immediately removed before write.)

---

## Mode: Refresh

For when a feature's `status:` or `affects:` frontmatter changed
(by hand or from another flow) and the per-repo notices are now
out of sync. Doesn't author a new feature — just re-renders
notices.

### Process

1. **Determine scope.** Either:
   - **All affected sub-repos:** scan `features/*.md`, collect the
     union of every open feature's `affects:` list. Each of those
     sub-repos gets its `active-features.md` re-rendered.
   - **One sub-repo:** user names the sub-repo; only that repo's
     notice is re-rendered.

2. **Render** each in-scope sub-repo's `active-features.md`
   wholesale from `features/*.md`, filtered by `affects:` and open
   `status:`, per "Notice file format" above.

3. **Show the rendered content** for each sub-repo. If a notice's
   entry list is empty (no remaining open features affect that
   repo), surface that the file will be deleted.

4. **Confirm.**

5. **Write.** For each sub-repo:
   - If the rendered entry list is non-empty, write the file.
   - If empty, delete the existing `active-features.md` (no empty
     notice files left behind).
   - Skip sub-repos whose `repos/<name>/` clone is missing; surface
     these to the user.

6. **Don't auto-commit.**

### When to use Refresh vs Open

- **Open** — when authoring a new feature.
- **Refresh** — when no new feature is being authored, but notice
  files need to be re-synced after manual frontmatter edits or
  cleanup.

If a feature is closing (status going to `shipped` or `abandoned`),
the user edits the frontmatter directly, then runs `/feature
refresh` — the closed feature drops off the notices automatically.

---

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
- **Don't write per-repo notices without user confirmation.** Even
  though the notice content is mechanically derived, the write is
  into another git repo — show before writing.
- **Don't combine `active-features.md` with `active-migrations.md`
  or any other concern.** One file per concern; the discipline is
  load-bearing.
- **Don't accumulate stale entries.** Notice body is regenerated
  wholesale every write. Closed features (`shipped`/`abandoned`)
  drop off automatically.
- **Don't leave empty notice files.** Delete instead.
- **Don't auto-commit.**

## When NOT to use this skill

- **Migration without new capability** — `/migration`
- **Single-repo feature** — file in sub-kit
- **Architectural decision** — `/decision`

## What "done" looks like

For **Open**:

- Feature plan drafted at `features/YYYY-MM-DD-short-name.md`.
- Any contract updates drafted alongside if obvious.
- Migration IDs listed under "Migrations included" but not opened.
- Per-repo `active-features.md` written for each affected sub-repo
  (or skipped with surfaced reason if the local clone is missing).
- Empty notices deleted rather than written empty.
- Shown to user, uncommitted.

For **Refresh**:

- Per-repo `active-features.md` re-rendered for each in-scope
  sub-repo from current `features/*.md` state.
- Empty notices deleted.
- Skipped sub-repos surfaced with reason.
- Shown to user, uncommitted.

In both cases: no auto-commit. The user commits.
