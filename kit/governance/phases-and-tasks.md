# Phases and tasks

The discipline behind what gets built, in what shape, in what order,
by whom, and how parallel work avoids stepping on itself.

The load-bearing claim: **the only thing that makes two phases run
in parallel without merge pain is disjoint code regions.** Process
discipline cannot rescue overlapping touch sets. So this doc starts
at the architectural prerequisite and works down.

---

## Vocabulary

Already defined in [`README.md`](README.md). Quick reminder:

```
feature   ──┐
            ├── crosses 2+ sub-repos; orchestrator-side artifact
phase     ──┐
            ├── per-sub-repo milestone; lives in tasks/PHASES.md
task      ──┘
            └── per-phase reviewable unit; lives in tasks/backlog/active/done
```

A **single-repo feature is just a phase.** Don't open a `/feature`
for it.

---

## Phase-shape rules

### P1 — A phase has one demoable outcome and one exit contract

If you can't write the outcome in one sentence — observable from
outside the system, not "we refactored X" — the phase isn't ready.
Examples that pass:

- "User can switch tenants without re-login."
- "All API responses include the `request_id` header."
- "The mobile app boots and renders the live transcript view."

Examples that fail:

- "Refactor the auth module" — internal, no outcome
- "Improve performance" — no observable bar
- "Various fixes" — not a phase, that's a chore PR

**Exit contract** = the conditions under which the phase is closed.
Always written outside-in:

- "Endpoint X returns shape Y for input Z, verified by test T."
- "Migration M has run on staging and on prod with zero errors."
- "Mobile build succeeds on iOS 17 and 18 with the new view."

"Tests pass" is not exit criteria. "These three specific tests pass
and they didn't exist before this phase" is.

### P2 — Phases run in parallel only when their code regions are disjoint

The mechanism, not just the rule:

1. **Up front, identify the shared seams** — DB schema, public API
   surfaces, event payloads, shared interfaces, generated types.
   These live in `bootstrap/contracts/` (orchestrator-side, cross-repo)
   or in `<sub>/docs/` (per-repo).
2. **A phase that needs to mutate a shared seam owns the seam change
   as its first task.** That task ships, the seam stabilizes, then
   downstream tasks under that phase or under other phases can build
   on it.
3. **A phase declares a freeze list** — files no concurrent phase
   may modify for the phase's duration. The freeze list is part of
   the phase entry in `tasks/PHASES.md`.
4. **If two candidate phases both need to mutate the same shared
   seam concurrently, they are not independent.** Choices:
   - Sequence them (phase A fully ships before phase B opens).
   - Merge them into one phase (both touch the seam; it's one
     coordinated change).
   - Split the seam (decide the two changes can be made independent
     by introducing a new layer — usually overkill, sometimes right).

**There is no fourth option that lets them run truly concurrent
without merge pain.** Plan accordingly.

### P3 — Phases declare dependencies; drafting and merging are different

Frontmatter on the phase entry (or on its task files):

```yaml
depends_on: [phase-id-or-task-id, ...]
affects: [sub-repo-name, ...]
freezes: [path/to/contract.md, path/to/seam.py]
```

- **Drafting** is always parallel. Two devs can plan two phases that
  depend on each other simultaneously — drafting doesn't touch code,
  doesn't take locks, doesn't conflict.
- **Merging** respects the DAG. A phase whose `depends_on` isn't yet
  merged to main does not merge to main.

The orchestrator, when authoring a `/feature`, lays out the
inter-phase DAG explicitly. Sub-kits resolve order locally within
their own repo's phases.

### P4 — Phase exit criteria are observable, fixed at open

Written when the phase opens. Not edited mid-phase except by an
explicit decision recorded in the phase entry's "Exit criteria
revisions" log. Pinning exit criteria is the only thing that keeps
"done" from drifting.

### P5 — Rollback path is part of phase open

What's the revert command? What data fixup is needed? What's the
canary or dark-launch story? If you can't write the rollback at
open, the phase isn't safe to merge. Often this also surfaces design
issues — a phase with no rollback path usually means the change is
more entangled than the spec admits.

### P6 — Phases are scoped to fit, not stretched to fill

A phase is "right-sized" when:

- Its tasks can ship in 1–3 weeks of dev time at sustainable pace.
- A reader of the phase entry can hold the whole thing in their
  head (1–2 screens of doc).
- The exit demo is a single coherent thing, not a list of unrelated
  capabilities.

If a phase grows past this, split it. Two coherent phases beat one
amorphous phase every time.

### P7 — A phase's tasks are listed in the phase entry, in order

`tasks/PHASES.md` shape (per claude-kit's existing convention,
extended by this governance):

```markdown
## Phase N: <noun phrase>

**Outcome:** <one sentence, outside-in>
**Exit criteria:** <bullets, each verifiable>
**Affects:** <sub-repos this phase touches; usually just this one>
**Depends on:** <phase IDs or "none">
**Freezes:** <files locked during this phase>
**Rollback:** <revert path>
**References:** <URLs, prior PRs, ADRs that informed this phase>

<2–4 sentence scope paragraph>

- TASK-NNN — <title>
- TASK-NNN — <title>
- ...
```

The task list is ordered. Order encodes intra-phase dependencies.

---

## Task-shape rules

### T1 — One task = one reviewable diff

Heuristic: if the diff exceeds ~300 lines or touches >3 unrelated
areas, split. Reviewer cognitive load is the metric, not LOC.

A task that grows mid-stream **splits, it doesn't bundle**. If you
discover three things while doing one, stop, file the other two,
finish the original. The bundle is the failure mode this rule
exists to prevent.

### T2 — A task is the smallest unit that proves a vertical slice works

Not a horizontal layer. Not a stub. Something a reviewer can run
end-to-end and confirm.

If the task is "add a backend endpoint with no caller," the next
task is "wire the caller and demonstrate end-to-end" — and the
first task should probably wait for the second to be planned, so
the API shape is right.

### T3 — Required sections in every task spec

See [`task-spec-shape.md`](task-spec-shape.md) for the full list.
Summary:

- **User request** (verbatim where possible)
- **Why** (the driver — if missing, task is premature)
- **Technical breakdown** (ordered, each step independently verifiable)
- **Acceptance criteria** (testable, line by line)
- **Out of scope** (what this task is not doing)
- **Contract impact** (links to seams or "none")
- **References** (URLs to docs, prior work, ADRs — saved at creation)
- **Definition of done** (tests, docs, AUDIT, advertisement update)

### T4 — Acceptance criteria must be testable

"Looks right" is not AC. Each AC is a thing a reviewer can run and
confirm. Examples:

- ✅ "Calling `POST /api/users` with payload P returns 201 and
  `{id, tenant_id}` matching P.tenant_id."
- ❌ "User creation works correctly."
- ✅ "Unit test `test_token_refresh_handles_expired` passes."
- ❌ "Tests pass."

### T5 — Tasks declare assumptions in writing

The dev, before coding, writes a one-paragraph restatement of the
task in their own words inside the task spec. The task-filer (or
orchestrator) confirms the reading is right before code lands.

This catches "I read it wrong" early. The cost is one round-trip;
the cost of finding out at code review is a re-do.

### T6 — Tasks declare the test before the code

"What would prove this works?" answered before writing the code.
Either as failing tests committed first (TDD), or as a manual-test
recipe written into the task spec. If the dev can't answer this
at task open, the spec isn't ready.

### T7 — Task dependencies are declared

Frontmatter:

```yaml
id: TASK-NNN
phase: <phase-id>
depends_on: [TASK-NNN, TASK-NNN]
blocks: [TASK-NNN]
```

The intra-phase order in `tasks/PHASES.md` encodes most dependencies;
explicit `depends_on` is for cases that cross phases or need to be
visible to readers who skip the phase entry.

### T8 — Tasks reference back to motivating artifacts

Every task spec body links the artifact that motivated it: a feature
plan, an ADR, a migration, a risk, an incident postmortem. If no
such artifact exists, the task is probably premature — open the
artifact first.

---

## Pre-planning rules

These apply to whoever drafts the task — orchestrator or sub-kit.

### PP1 — Read the actual code, not your model of it

Walk the relevant files. Note the current state. Link the files in
the task's References. No "I think this is how it works."

### PP2 — Read official docs and save the URL

Every external dependency or framework feature touched gets a URL
in References. "Based on training data" is not acceptable backing.

### PP3 — Check whether the work is cross-cutting before drafting

Cross-cutting (2+ sub-repos) → write a `/feature` plan first; let
the sub-kits draft tasks under it. Single-repo → file phases/tasks
directly.

### PP4 — Read existing migrations, features, ADRs

`/sync-check`, the per-repo state file at `state/sub-repos/<name>.md`,
and any open `features/<x>.md` matching this work. Don't draft a
task into a contradiction.

### PP5 — Check for in-flight work that conflicts

Read `<sub>/tasks/active/`. If your phase's freeze list overlaps
something already in flight, the new phase waits or revises scope.

---

## Branch and PR conventions under this governance

Already established in [`claude-kit-reference.md`](../claude-kit-reference.md)
"Branch / PR conventions in claude-kit":

- **Sub-kit task branches:** `task/TASK-XXX-short-slug`
- **Sub-kit PR titles:** `TASK-XXX: <task title>`
- **Orchestrator-authored task spec drops:** `chore/orch-task-<NNN-slug>`
- **Orchestrator non-running file edits:** `chore/orch-<YYYY-MM-DD-slug>`

Governance addition:

- **Phase-introduction PRs** (the orchestrator or sub-kit adding a
  new phase to `tasks/PHASES.md` and `tasks/ROADMAP.md`) use
  `chore/orch-phase-<id>` or `chore/phase-<id>` respectively. The
  PR body links the motivating feature plan or ADR.
- **Phase-spanning task batches** are not a thing. Each task is its
  own PR. If a phase has 8 tasks, that's 8 PRs (plus the phase-
  introduction PR).

---

## Conflict and overlap policy

When two phases discover mid-execution that they need the same file:

1. **Stop the later one.** First-in-flight phase keeps its freeze.
2. **Decide:** does the later phase's scope still hold without the
   contended change, or does it need to wait?
3. **If wait:** the later phase pauses; record the decision in its
   phase entry; resume when the first phase merges.
4. **If revise scope:** the later phase's spec is updated, the
   contended change is dropped or deferred to a follow-up phase.

The discipline is: **freeze lists win.** The phase that declared
the freeze first holds it. This forces freeze lists to be honest
at phase open — overclaim and you block real work; underclaim and
you collide mid-stream.

---

## When this governance is followed correctly

- Two devs in two phases in two sub-repos, or two phases in the same
  sub-repo, can both ship to main on the same day without merge
  rework.
- A reviewer reading a task spec cold can run the acceptance criteria
  and verify them.
- When a phase needs to roll back, the rollback is a documented
  command, not an improvisation.
- The orchestrator's `/feature` plan, the sub-kit's `tasks/PHASES.md`
  entry, and the sub-kit's `tasks/backlog/TASK-NNN.md` are all
  legible to each other — same vocabulary, same shape rules, same
  required sections.

When it's not, the failure mode is the obvious one: scope creep,
merge conflicts, "wait, what was this task supposed to do?" at PR
review, no rollback path when something breaks in prod.

---

## See also

- [`task-spec-shape.md`](task-spec-shape.md) — the required sections
  in detail
- [`dev-preflight.md`](dev-preflight.md) — what the dev does before
  writing code
- [`../sub-projects.md`](../sub-projects.md) — how the orchestrator
  writes into sub-repos (write protocol, code boundary)
- [`../claude-kit-reference.md`](../claude-kit-reference.md) — the
  sub-kit's own structure and discipline
- [`../skills/feature/SKILL.md`](../skills/feature/SKILL.md) — how
  cross-cutting features are authored
- [`../skills/tasks/SKILL.md`](../skills/tasks/SKILL.md) — how
  active tasks are aggregated across sub-repos
- [`../../proposals/README.md`](../../proposals/README.md) +
  [`../skills/propose/SKILL.md`](../skills/propose/SKILL.md) +
  [`../skills/promote/SKILL.md`](../skills/promote/SKILL.md) —
  staging area for phases and tasks before they're committed to
  the sub-repo. Apply the rules in this doc to proposed phases
  just as to committed ones; the staging area is for iterating
  on the rules' inputs (outcomes, exit criteria, freeze lists,
  task lists) before commitment.
