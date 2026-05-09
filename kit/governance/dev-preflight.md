# Developer pre-flight

The checklist a developer (human, sub-kit's claude, or any future
agent) runs through **before writing code on a task**.

Each item below has a *what* and a *why*. The why matters — knowing
why the rule exists lets you handle edge cases instead of blindly
following the rule.

This applies to anyone working a task drafted under this governance.
The orchestrator never executes code, but it follows steps 1–5 when
drafting task specs (so the spec it produces is itself ready for a
doer to consume).

---

## The checklist

### 1. Read the task spec end to end

Then **restate it in writing in the spec's `## Assumptions`
section** in your own words. Confirm with the task-filer (or with
the spec itself if the AC is precise enough) before proceeding.

**Why:** the highest-frequency early-stage failure is "I built the
wrong thing because I read the spec wrong." The fix is one round
trip. Skipping this step trades a 1-minute confirmation for a
multi-hour rebuild.

### 2. Read the active orchestrator notices for this repo

`<sub-repo>/.claude/active-*.md`. As of v1, that's:

- `active-migrations.md` — open cross-repo migrations
  affecting this repo
- (more concerns may exist over time — read all `active-*` files)

Then read `<sub-repo>/.claude/shared/inbox.md` for any messages
the orchestrator left for this repo, and `shared/notes.md` for
recent durable context.

**Why:** the orchestrator may have already flagged something the
task spec doesn't mention — a migration in flight that constrains
your change, a freeze on a contract you were about to touch, a note
about a gotcha discovered last week. Reading first costs a minute
and saves you from contradictions.

### 3. Read the related code

Not just the file you're editing. Also:

- Its callers (`grep` for the function name)
- Its tests
- The data structures it consumes / produces
- Adjacent files in the same module

Note what you found in the task spec's `## Notes for the reviewer`
section as you go — it's free documentation for the eventual
reviewer.

**Why:** your model of the code is wrong. Always. The cost of being
wrong is debugging from a false premise. The cost of reading first
is 5–15 minutes.

### 4. Read the references

Every URL in the task spec's `## References` section. Every linked
ADR, migration, prior PR. If any reference is broken, missing, or
outdated, that's a gap — fix the spec or flag the gap to the
task-filer before coding.

**Why:** a task without grounded references is a task built on
hallucinations. A 60-second skim of an official doc page beats 30
minutes of re-deriving behavior from API responses.

### 5. Read the orchestrator-side context

If the task is connected to orchestrator-side artifacts (and most
non-trivial ones are):

- The relevant `bootstrap/contracts/<file>` if the task has
  contract impact
- The relevant `bootstrap/conventions/<file>` for naming, error
  handling, auth
- Any `decisions/<id>.md` (ADR) that bears on this work
- The per-repo state file at `state/sub-repos/<this-repo>.md`,
  especially the **Notes** section
- Any open `features/<x>.md` that lists this repo in `affects`

**Why:** macro-level decisions live in the orchestrator. The sub-kit
that ignores them re-decides things that were already decided, and
ships changes inconsistent with sister repos.

### 6. Plan the steps in writing

Update the task spec's `## Technical breakdown` section with
numbered steps. Each step has:

- A clear action ("Add column", "Update model", "Add test")
- A verification ("Migration applies", "Test passes", "Endpoint
  returns 201")

If you can't write the verification, the step is too coarse — split
it. Plan goes into the task file **before code lands** — it's part
of the task artifact, not a private scratch pad.

**Why:** writing the plan forces the missing details to surface. A
step you can't write a verification for is a step you don't
understand yet.

### 7. Identify the smallest vertical slice that proves the path works

Build that first, end-to-end. Then widen.

If the task is "add an endpoint and use it from the iOS client,"
the vertical slice is "endpoint + a single iOS call site rendering
a hardcoded result, end to end." After that works, widen.

**Why:** a horizontal layer (e.g., "build the whole API surface,
then build the whole client") fails late and ambiguously. A vertical
slice that compiles and runs proves the whole stack is sound. When
something breaks, you know where because the slice is small.

### 8. Write the test before (or alongside) the code

If the project's testing convention permits TDD, write the failing
test first. Otherwise, at minimum, write the manual-test recipe in
the task spec before writing code.

**Why:** acceptance criteria are aspirational until something can
verify them. Writing the test surfaces ambiguity in AC: if you
can't write the test, the AC isn't actually testable, and that's
a spec issue to fix before coding.

### 9. Write the code

With the plan and tests visible. Don't write past the plan without
updating the plan. If you discover a step the plan didn't anticipate:

- Update the plan first.
- Note in `## Notes for the reviewer` that the plan changed and
  why.
- Then continue.

**Why:** the plan is a contract with your future self and the
reviewer. Writing past it silently is how scope creeps.

### 10. Self-review against acceptance criteria, line by line

For each AC item: how do I know this is satisfied? What did I
verify? Check it off only when there's a real verification (a test
ran, an endpoint was hit, a log line confirmed the behavior).

**Why:** AC is the contract. "I think it works" is not satisfaction.

### 11. Self-review the diff for unintended changes

`git diff` against main, top to bottom. Anything outside the task's
stated scope:

- Reverts (preferred)
- Splits into a separate task (if the change is real and worth
  shipping but doesn't belong here)

**Why:** scope creep at PR review costs the reviewer their time
and you a round-trip. Catching it during self-review is free.

### 12. Update the audit trail

- Move the task file: `git mv tasks/active/TASK-NNN.md
  tasks/done/TASK-NNN.md`
- File the AUDIT entry in `<sub>/tasks/AUDIT.md` per claude-kit's
  format
- Update `<sub>/.claude/active.md` (advertisement) — current task
  → next task or idle
- If the task closes a migration step, update the
  `active-migrations.md` entry's "This repo's part" from 🟡 → ✅
- If durable context emerged that future devs should know, append
  to `<sub>/.claude/shared/notes.md` or `shared/architecture.md`

**Why:** the next session — yours or someone else's — depends on
this state being current. An out-of-date advertisement and a stale
AUDIT log are how the orchestrator's understanding drifts from
reality.

---

## Common failure modes this checklist defends against

| Failure | Step that catches it |
|---|---|
| "I built the wrong thing" | 1 (assumptions confirmed in writing) |
| "I conflicted with an in-flight migration" | 2 (read active notices) |
| "There was a caller I didn't see" | 3 (read related code) |
| "I implemented the framework wrong because I assumed its API" | 4 (read references) |
| "My change conflicts with a contract change" | 5 (read orchestrator-side context) |
| "I got 80% done before realizing the design was wrong" | 6, 7 (plan + vertical slice) |
| "The AC isn't actually testable" | 8 (test before code) |
| "I built more than the task asked for" | 11 (self-review for scope) |
| "The orchestrator's state is wrong about my repo" | 12 (audit trail) |

---

## When this is being run by claude (sub-kit or otherwise)

The orchestrator's role: when it drafts a task spec into
`<sub>/tasks/backlog/`, the spec must already include the references
and contract-impact analysis the doer needs (steps 4 and 5 inputs).
The doer still does the read — the orchestrator's job is to make
sure the read targets are listed, not to outsource the reading.

The sub-kit's role (per its `task-rules.md`): the sub-kit's claude
runs this checklist on every task before code, and is responsible
for steps 1–12.

The human's role: optional, but the same checklist works for hand-
worked tasks. The checklist is the discipline; the actor is who
applies it.

---

## See also

- [`phases-and-tasks.md`](phases-and-tasks.md) — what governs the
  task you're doing in the first place
- [`task-spec-shape.md`](task-spec-shape.md) — the sections of the
  spec you'll be reading and writing into
- [`../sub-projects.md`](../sub-projects.md) — the orchestrator's
  write protocol and code boundary
- [`../templates/sub-repo-shared/README.md`](../templates/sub-repo-shared/README.md)
  — the durable two-way per-repo channel referenced in step 2
