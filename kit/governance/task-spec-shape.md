# Task spec shape

What every task spec drafted under this governance must contain.
Extends claude-kit's `task-template.md` — adds required sections,
makes existing ones more rigorous.

A task spec is the contract between the task-filer (orchestrator,
sub-kit, or human) and the task-doer (whoever picks the task up
later). The doer should be able to act on the spec **cold**, with
no further conversation. If they can't, the spec is incomplete.

---

## File location and naming

```
<sub-repo>/tasks/backlog/TASK-NNN-short-slug.md   # not yet active
<sub-repo>/tasks/active/TASK-NNN-short-slug.md    # in progress
<sub-repo>/tasks/done/TASK-NNN-short-slug.md      # shipped
```

Sub-kit transitions via `git mv`. ID format: `TASK-NNN` where NNN is
zero-padded to 3 digits, monotonic across the sub-repo's history.

### Pre-promotion: orchestrator-side staging

Tasks may be drafted, iterated, and refined in the orchestrator's
staging area before they land in the sub-repo. See
[`../../proposals/README.md`](../../proposals/README.md) and the
[`/propose`](../skills/propose/SKILL.md) /
[`/promote`](../skills/promote/SKILL.md) skills.

Staged tasks live at `proposals/<repo>/backlog/<slug>.md` with the
same body shape as below — required sections are identical. The
proposal frontmatter has additional fields for proposal management
(slug, target_repo, target_phase, status, proposed_at, etc.) that
get stripped at promotion. The body is preserved verbatim.

Recommended path: stage non-trivial tasks via `/propose`, iterate
until ready, then `/promote`. Direct-drop into `<sub>/tasks/backlog/`
is fine for tasks that are fully-formed in one session.

---

## Required frontmatter

```yaml
---
id: TASK-NNN
title: <descriptive, matches filename slug>
phase: <phase-id from tasks/PHASES.md>
status: backlog | active | done
depends_on: [TASK-NNN, ...]   # tasks that must merge before this can start
blocks: [TASK-NNN, ...]       # tasks that wait on this
affects: [<files or modules this touches>]
contract_impact: yes | no     # see "Contract impact" section below
references:                    # also expanded in body
  - <url-or-path>
filed_by: <orchestrator | sub-kit | human-handle>
filed_at: YYYY-MM-DD
---
```

Most fields can be empty arrays / "no" / "none" — the absence is
informative, the missing field is not.

---

## Required body sections

In this order. Headings exactly as below.

### `## User request`

The human ask, verbatim where possible. If paraphrased, mark with
*(paraphrased)*. If the request came from a feature plan or ADR,
quote the relevant paragraph and link the artifact.

If there's no traceable user request behind this task, the task
is probably premature. Stop and ask: who wants this and why?

### `## Why`

The driver. One paragraph. What does the world look like after this
task ships that didn't before? What problem does it solve, what
metric does it move, what risk does it close?

If this section reduces to "because the spec said so," go up the
chain — read the feature plan or ADR — and bring the real why down
into this task.

### `## Assumptions`

The doer's restatement of the task in their own words.

> "I read this as: I'm adding a `tenant_id` column to the `users`
> table, backfilling from the existing `org_id`, and updating the
> `User` model and the three callers in `auth.py`, `admin.py`, and
> `signup.py`. I'm NOT touching the iOS or web side; that's a
> separate phase."

The task-filer or orchestrator confirms this reading before code
lands. Catches "I read it wrong" for the cost of one round-trip.

### `## Technical breakdown`

Numbered, ordered steps. Each step has a verification:

```
1. Add `tenant_id` column to `users` table via Alembic migration.
   Verify: `alembic upgrade head` succeeds against a clean DB;
   `\d users` in psql shows the column.

2. Backfill existing rows: `UPDATE users SET tenant_id = org_id`.
   Verify: `SELECT COUNT(*) FROM users WHERE tenant_id IS NULL` → 0.

3. Update `User` model in `models/user.py` to include `tenant_id`.
   Verify: `pytest tests/models/test_user.py` passes.

...
```

Steps must be **independently verifiable**. If a step's verification
is "we'll see when we test the whole thing," the step is too coarse;
split it.

### `## Acceptance criteria`

Checklist. Each item is a thing a reviewer can run and confirm.

```
- [ ] Migration `2026_05_08_tenant_id_on_users` applies cleanly on
      a fresh DB.
- [ ] Migration applies cleanly on a snapshot of production data
      (run against staging first).
- [ ] `users` table has a non-null `tenant_id` column for every row.
- [ ] `User.tenant_id` is accessible on every User instance loaded
      via `User.query.first()`.
- [ ] Test `test_user_has_tenant_id` passes (added in this task).
- [ ] Existing tests in `tests/models/test_user.py` all pass.
```

**Anti-patterns** to reject:

- "Code looks correct"
- "No regressions"
- "Performance is acceptable" (unless followed by a measurable bar)
- "Documentation is updated" (link the doc; specify what changed)

### `## Out of scope`

What this task is **not** doing. Defends against scope creep
mid-stream.

```
- Updating the iOS client to pass `tenant_id` in requests
  → that's TASK-073 in the ios repo's phase 4
- Adding a UI for changing tenants
  → not in this phase; tracked in features/2026-05-tenant-switching.md
- Refactoring the `org_id` column to be removed
  → deferred; tracked as phase-5 followup
```

Without this section, "while I'm in there" expansions sneak in and
the diff bloats. With it, the doer has explicit permission to defer.

### `## Contract impact`

Does this task touch:

- A public API endpoint shape?
- A DB schema?
- An event payload?
- A cross-repo interface (gRPC, REST, GraphQL, message bus)?
- A generated type or shared library?

If yes, **link the contract file** in `bootstrap/contracts/` (or the
sub-repo equivalent). The doer reads the contract before writing
code; the reviewer checks the contract is updated alongside.

If no, write "None" and move on. Don't omit the section — its
presence (even with "None") proves the question was asked.

### `## References`

URLs and paths to:

- Official docs for any framework / library / tool the task uses.
  Saved at creation, not as an afterthought.
- Prior PRs that touched the same area.
- ADRs that constrain how this is built.
- Migration files relevant to this change.
- Sub-repo `docs/` runbooks if applicable.
- Orchestrator artifacts: feature plan, risk file, incident
  postmortem.

```markdown
- Alembic docs: https://alembic.sqlalchemy.org/en/latest/autogenerate.html
- Prior column-add PR: https://github.com/.../pull/231
- ADR: <sub>/docs/decisions/2026-04-tenant-isolation.md
- Feature plan: features/2026-05-tenant-switching.md
- Migration template: kit/templates/sub-repo-notices/migrations.md.template
```

### `## Definition of done`

Beyond AC — the task is "done" only when:

- All acceptance criteria are checked.
- Tests for the new behavior exist (per the project's test pairing
  rule from `<sub>/CLAUDE.md`).
- Doc updates are in the same PR (if the change affects user-facing
  or operator-facing docs).
- AUDIT entry filed in `<sub>/tasks/AUDIT.md` per claude-kit's
  closing-report discipline.
- Advertisement file `<sub>/.claude/active.md` updated (current task
  → idle, or → next task).
- If the task is part of a migration, the migration's
  `<sub>/.claude/active-migrations.md` entry updates from
  ⚪/🟡 → ✅ for this repo.

---

## Optional body sections

### `## Test plan`

If the test approach is non-obvious or needs more than the AC
section can carry. Examples: a fuzz strategy, a load test setup,
a multi-environment validation.

### `## Notes for the reviewer`

Things the doer wants the reviewer to look at first, gotchas
discovered during the work, etc. Filled in as the task progresses,
not at filing.

### `## Closing report`

Per claude-kit's closing-report discipline (see
[`../claude-kit-reference.md`](../claude-kit-reference.md) "Closing
report"). Filled in when the task moves to `done/`. Many sub-kits
put this in the PR body instead; either is fine, as long as it
exists somewhere durable.

---

## Anti-shapes to refuse

If a task spec drops in your lap missing any of these, push back
before writing code:

| Missing | What to ask |
|---|---|
| Why | "Who wants this and what changes when it ships?" |
| Acceptance criteria | "How will the reviewer know this is done?" |
| Assumptions | "Let me restate this — is that right?" |
| Out of scope | "What's adjacent that I should *not* do?" |
| Contract impact | "Does this touch any shared interface?" |
| References | "What docs / prior PRs should I read first?" |

A task without these isn't a task — it's a wish.

---

## See also

- [`phases-and-tasks.md`](phases-and-tasks.md) — phase shape and
  parallel-execution rules
- [`dev-preflight.md`](dev-preflight.md) — what the doer does before
  writing code
- [`../claude-kit-reference.md`](../claude-kit-reference.md) — the
  sub-kit's task system layout
