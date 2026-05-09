---
name: migration
description: Open, update, list, or close a cross-repo migration. A migration is a coordinated state transition across sub-repos with a defined start, blast radius, and close criteria. Triggered by "/migration", "open a migration", "track this change", "we're changing the schema", "this affects multiple repos", "let's coordinate this", "close the migration".
---

# /migration — Cross-repo migration coordinator

The operational core of the orchestrator after the audit settles. Most
ongoing CTO work is opening, watching, and closing migrations.

A **migration** is a coordinated state transition across sub-repos with
a defined start, blast radius, and close criteria. See
`migrations/README.md` for the full lifecycle.

**Output pattern:** [Pattern 4 — Sprint task board](../../output-catalogue.md#4--sprint-task-board)
for `list` mode (per-migration row with per-repo state symbols) +
[Pattern 24 — Comparison matrix](../../output-catalogue.md#24--comparison-matrix)
when surfacing per-repo state across migrations +
[Pattern 1 — Hero completion card](../../output-catalogue.md#1--hero-completion-card)
on `close`. Rendered migration files follow `migrations/_template.md`
markdown structure (durable artifact).

## Modes

This skill has four modes. Detect which from user input:

- **Open** — "open a migration", "we're changing X", "track this"
- **Update** — "update migration X", "api is done", "mark ios merged"
- **List** — "what's open", "show migrations", "any stale ones"
- **Close** — "close migration X", "X is done"

If ambiguous, ask which.

---

## Notice file format

This skill is the only orchestrator → sub-repo write path in v1
(see `kit/templates/sub-repo-notices/README.md`). When writing into
a sub-repo's `.claude/active-migrations.md`:

- **Use the template at
  `.claude/templates/sub-repo-notices/migrations.md.template`** in
  this orchestrator instance. It defines the header (auto-managed
  warning + source-orchestrator pointer + timestamp), the body
  shape, and the comment hint inside `## Open` describing each
  entry's structure.
- **Substitute placeholders** when rendering:
  `{{ORCHESTRATOR_PATH}}` — absolute path to this orchestrator
  instance; `{{REPO_NAME}}` — the sub-repo name from
  `state/manifest.md`; `{{TIMESTAMP}}` — ISO date+time at write
  time; `{{SKILL}}` — `/migration`; `{{MIGRATION_ENTRIES}}` — the
  rendered entry list.
- **Regenerate the body wholesale** every time. Don't read the
  existing file and merge — that's how stale entries accumulate.
  Read `migrations/active/`, filter for migrations whose `affects`
  list contains this repo, render entries in directory order
  (newest first by ID date prefix).
- **Delete the file when the entry list is empty.** No empty
  notice files left behind.
- **Don't invent new sections.** If the format needs to change,
  update the template in the kit, then `/sync` instances.

The same pattern will apply to future concerns
(`active-adrs.md`, etc.) — see
`kit/templates/sub-repo-notices/README.md` for adding new ones.

---

## Mode: Open

### Behavior contract

- **Confirm blast radius before writing.** Determine which sub-repos
  are affected; show the list to the user; get confirmation.
- **Update the contract file in the same operation.** A migration that
  changes contracts must update the relevant `contracts/*.md` file.
  Don't open the migration without it (or without an explicit "the
  contract update is in this commit: <ref>" note).
- **Write the sub-repo notice files only after user confirmation.**
  Each affected sub-repo gets `.claude/active-migrations.md` updated.
  This is a write into another git repo — confirm before doing it.

### Process

1. **Surface the change.** Ask the user:
   - One-sentence "what's changing"
   - Which sub-repos this affects (if not obvious)
   - The driver — why now, why this shape
   - Whether contract files need updating, and which

2. **Generate the migration ID.** `YYYY-MM-DD-short-name`. Today's
   date, kebab-case slug derived from the change.

3. **Draft the migration file** at `migrations/active/<id>.md` from
   `migrations/_template.md`. Fill in:
   - Frontmatter (id, opened, status: in-progress, affects,
     contracts_changed, related_adrs)
   - "What changed" — technical specifics
   - "Why" — driver
   - "Per-repo state" — every affected repo as ⚪ not started
   - "Plan" sections for each affected repo
   - "Validation" checklist
   - "Sequence / dependencies" if order matters
   - "Rollback" path or note

4. **Update contract files.** If the migration changes any
   `contracts/*.md`, draft those edits in the same operation. Show all
   the diffs to the user.

5. **Draft sub-repo notices.** For each affected sub-repo:
   - The sub-repo's working tree is at `repos/<name>/` (convention).
     Notice writes require a local clone. If `repos/<name>/` doesn't
     exist, surface that to the user in step 6 — they can run
     `bin/setup` to clone it or skip the notice for that sub-repo.
   - Render `repos/<name>/.claude/active-migrations.md` content
     **wholesale** from `migrations/active/`, using the template at
     `.claude/templates/sub-repo-notices/migrations.md.template` —
     see "Notice file format" above. The new migration is now in
     `migrations/active/`, so it lands in the rendered output
     naturally; older still-open migrations affecting this repo
     also stay in.
   - Don't write yet — show the rendered content as part of step 6.

6. **Show everything.** All drafted files (migration, contract updates,
   per-repo notices) shown together. Get user confirmation.

7. **Write.** On confirmation, write all files. Don't auto-commit; the
   user reviews and commits.

8. **Confirm.** Show the migration ID, the affected files, and the
   suggested next moves (file per-repo tasks in each sub-kit).

### What to ask if unclear

- "What's the smallest accurate description of this change?"
- "Which sub-repos must touch this?" — if user says "all", probe; not
  every change actually affects all
- "Is there a deploy ordering constraint?" — informs the sequencing
  section
- "What proves this worked end-to-end?" — drives the validation
  checklist

---

## Mode: Update

### Behavior contract

- **Read the current file before editing.**
- **Update the per-repo state line.** Use ⚪ → 🟡 → ✅. Be specific:
  "merged in PR #234", "deployed to staging", "deployed to prod".
- **Mark validation items only when verified.** "iOS reads tenant_id
  without crash" is checked when there's evidence (test passed,
  manual verification, deployed and observed). Not just because the
  code merged.

### Process

1. Identify the migration (by ID or by recent context).
2. Read it.
3. Apply the update — usually a per-repo state change or a validation
   checkbox.
4. Show the diff. Confirm.
5. Write.

### Auto-promote to validating

If after the update **all per-repo state is ✅** but **validation has
unchecked items**, change the frontmatter `status` from `in-progress`
to `validating`. Tell the user explicitly: "all per-repo work is
done; now in validation phase. <N> items remain."

---

## Mode: List

### Behavior contract

- **Read all of `migrations/active/`.**
- **For each, render a one-line summary** with status, affects, and
  any drift signal.
- **Flag stale migrations** — opened ≥14 days ago with at least one
  per-repo state still ⚪ or 🟡.
- **Don't show closed migrations** unless asked.

### Process

1. List files in `migrations/active/`.
2. For each, parse frontmatter and per-repo state lines.
3. Render as a table or list:

   ```
   open migrations:

   - 2026-05-07-user-add-tenant-id  [in-progress]  api ✅ ios 🟡 web ⚪ devops ✅
   - 2026-04-30-auth-token-format-v2 [validating]  api ✅ ios ✅ web ✅ devops ✅  · 3 validation items remain
   - 2026-03-01-events-rename       [in-progress]  api ✅ ios ⚪ web ⚪ devops ✅  ⚠ STALE — 67 days, 2 repos not started
   ```

4. If any stale, surface them at the top with the staleness reason.

---

## Mode: Close

### Behavior contract

- **All per-repo states must be ✅.** Refuse to close otherwise.
- **All validation items must be checked.** Refuse otherwise.
- **The user must confirm.** Closure is deliberate.
- **Move the file** from `active/` to `closed/`. Update frontmatter
  `status: closed` and add `closed: YYYY-MM-DD`.
- **Remove the migration from each affected sub-repo's
  `.claude/active-migrations.md`.** If no migrations remain, delete
  the file.

### Process

1. Identify the migration.
2. Read it. Verify per-repo state and validation are complete. If not,
   show what's missing and stop.
3. Update frontmatter (`status: closed`, add `closed: YYYY-MM-DD`).
4. Move file to `migrations/closed/<id>.md`.
5. For each affected repo: regenerate `.claude/active-migrations.md`
   from `migrations/active/` (this migration is now in `closed/`,
   so it drops off automatically). Use the template per "Notice
   file format" above. If the rendered entry list is empty, delete
   the file instead of writing an empty one.
6. Append a one-liner to `roadmap.md` "Recently shipped" section.
7. Show the user the closure summary.
8. Don't auto-commit.

---

## Style rules

- **No emoji theater.** ⚪ 🟡 ✅ ⚠ are load-bearing for status. Don't
  sprinkle others.
- **Migration titles in kebab-case slug + descriptive ID.** The ID is
  the filename; the title in the body is human-readable.
- **Show diffs, don't summarize them.** When proposing edits, show the
  actual file content (or a clear diff). Confirmations off summaries
  are how silent drift happens.

## What you must NOT do

- **Don't auto-close migrations.** Even if all states look ✅,
  confirmation is required.
- **Don't open a migration that doesn't cross repos.** Single-repo work
  is a sub-kit task, not a migration.
- **Don't write to a sub-repo without user confirmation,** including
  for the `.claude/active-migrations.md` notice. That's still a write
  into someone else's repo.
- **Don't merge two migrations into one.** If they're coupled, that's
  one migration; if they're independent, they stay separate.

## When NOT to use this skill

- **Filing an ADR** — use `/decision`.
- **Authoring a cross-cutting feature spec** without an active state
  transition — use `/feature`.
- **Reading current state** — use `/status` or `/sync-check`.
- **Per-repo task work** — that's the sub-kit's job.

## What "done" looks like

For each mode:

- **Open:** migration file, contract updates, per-repo notice files all
  drafted and either written (with confirmation) or surfaced for review.
- **Update:** the migration file's relevant lines updated, status
  auto-promoted to validating if applicable.
- **List:** a clear summary of open migrations with stale items
  flagged.
- **Close:** migration moved to `closed/`, sub-repo notices cleaned up,
  roadmap entry added.

In all cases: no auto-commit. The user commits.
