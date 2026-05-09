# Governance

The discipline contracts the orchestrator follows when it drafts
cross-cutting features, drops task specs into sub-repos, and reads
sub-kit state. Sub-kits should mirror these rules; humans working
in any repo should know them.

These docs are **rules**, not skills. Skills implement them;
contracts in [`sub-projects.md`](../sub-projects.md) reference them.

## Files

| File | Audience | What it covers |
|---|---|---|
| [`phases-and-tasks.md`](phases-and-tasks.md) | orchestrator + sub-kit + humans | The shape of phases and tasks: what makes them parallelizable, how dependencies are declared, how big a task should be, what every task spec contains. |
| [`task-spec-shape.md`](task-spec-shape.md) | orchestrator + sub-kit | Required sections in any task spec drafted under this governance. Extends claude-kit's `task-template.md`. |
| [`dev-preflight.md`](dev-preflight.md) | sub-kit + humans | The checklist a developer (human or sub-kit's claude) follows before writing code on a task. |

## Glossary

- **Feature** — a cross-cutting capability that spans 2+ sub-repos.
  Authored by the orchestrator via [`/feature`](../skills/feature/SKILL.md);
  decomposes into phases in each affected sub-repo.
- **Phase** — a milestone within one sub-repo with a single demoable
  outcome and an exit contract. Lives in `<sub>/tasks/PHASES.md`.
  Phases that touch disjoint code regions can run in parallel;
  phases that share seams are sequenced.
- **Task** — the smallest reviewable unit of dev work, typically one
  PR. Lives in `<sub>/tasks/backlog/` → `active/` → `done/`. Every
  task ships under exactly one phase.
- **Seam / shared seam** — a code or contract surface multiple phases
  might want to mutate (DB schema, public API, event payload, shared
  interface, generated types). Seams are the things `bootstrap/contracts/`
  exists to track.
- **Freeze list** — files a phase declares as locked for its
  duration. Concurrent phases may not modify items on each other's
  freeze lists.
- **Disjoint code regions** — non-overlapping file sets and non-shared
  seams. Two phases are independent only when their changes are
  disjoint by this definition; otherwise they're sequenced.
- **Active-`<concern>`** — auto-managed notice file the orchestrator
  drops at `<sub>/.claude/active-<concern>.md`. Regenerated wholesale,
  deleted on empty. See [`templates/sub-repo-notices/`](../templates/sub-repo-notices/).
- **Shared context** — durable, hand-edited per-repo memory at
  `<sub>/.claude/shared/`. Both orchestrator and sub-kit read and
  write. Never auto-regenerated. See
  [`templates/sub-repo-shared/`](../templates/sub-repo-shared/).

## Where governance applies

| Action | Rules apply? |
|---|---|
| Orchestrator drafting a `/feature` plan | yes — feature plan must list per-repo phases that satisfy the phase-shape rules |
| Orchestrator drafting a task spec into `<sub>/tasks/backlog/` | yes — task spec must satisfy `task-spec-shape.md` |
| Orchestrator adding a phase to `<sub>/tasks/PHASES.md` | yes — phase entry must include declared seams, freeze list, exit criteria |
| Sub-kit creating its own tasks/phases | yes — the sub-kit is bound by the same governance |
| Human creating a task by hand | yes — the human is bound by the same governance |
| Orchestrator updating a doc (ADR, conventions, runbook) | no — that's a doc edit, not task work |

## What governance does NOT cover

- **How the sub-kit ships code** — that's claude-kit's `task-rules.md`
  (verification gate, closing report, AUDIT entries).
- **The code boundary** — the orchestrator's "don't touch running
  files without override" rule lives in
  [`sub-projects.md`](../sub-projects.md) "Code boundary."
- **Cross-repo migrations** — those have their own skill ([`/migration`](../skills/migration/SKILL.md))
  and template ([`migrations.md.template`](../templates/sub-repo-notices/migrations.md.template)).
  Migrations interact with phases (a phase may include a migration)
  but the migration's discipline is separate.
