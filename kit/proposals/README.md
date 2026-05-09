# Proposals — orchestrator-side staging area

Per-repo holding area for **proposed phases and tasks** before they
land in a sub-repo's actual task system. Lets the CTO plan, draft,
iterate, and refine across multiple repos in parallel without
opening PRs in those repos until the work is ready to commit.

The principle: **draft cheaply, promote deliberately**. PR churn
in sub-repos is a tax on contributors; this directory is where
churn happens before commitment.

---

## Directory layout

```
proposals/
├── README.md                       # this file (synced from kit)
├── _template-task.md               # task proposal body template
├── _template-PHASES.md             # PHASES.md template (per-repo first-time)
└── <repo>/                         # one directory per registered sub-repo
    ├── PHASES.md                   # proposed phase definitions for this repo
    ├── backlog/                    # tasks being drafted / iterated
    │   └── <slug>.md
    ├── promoted/                   # archive — tasks promoted to <sub>/tasks/
    │   └── <slug>.md
    └── retired/                    # archive — tasks abandoned without promotion
        └── <slug>.md
```

Per-repo subdirectories are created lazily by `/propose new` on
first use for a given repo. Empty directories don't get created
preemptively at registration.

---

## Proposal lifecycle

```
[user idea]
   │
   ▼
proposals/<repo>/backlog/<slug>.md   ◄─── /propose new
   │  (drafted, iterated, refined)
   │
   ├─── /propose update              (edit in place)
   │
   ├─── /propose retire              (move to retired/)
   │     │
   │     ▼
   │   proposals/<repo>/retired/<slug>.md
   │
   └─── /promote                     (open PR in sub-repo)
         │
         ▼
       <sub-repo>/tasks/backlog/TASK-NNN-<slug>.md
         │
         ▼
       proposals/<repo>/promoted/<slug>.md   (archive with promotion record)
```

Once promoted, the sub-kit owns the task. The orchestrator's job is
done; the archive in `promoted/` is just for audit ("when did this
get promoted, what TASK-NNN was assigned, what PR opened it").

---

## Proposal vs the existing direct-drop path

[`sub-projects.md`](../.claude/sub-projects.md) "Write protocol #3
— Coding changes — task spec drafted into `<sub>/tasks/backlog/`"
already supports drafting a task spec directly into a sub-repo via
PR. That path stays. Use it when:

- The task is fully formed in one session, no iteration needed.
- You want the task visible in the sub-repo's PR list immediately.

Use the proposal path when:

- You're planning multiple tasks across multiple repos and want to
  see them together before committing.
- You expect to iterate on the spec across multiple sessions.
- You're laying out phases that aren't ready for the sub-repo's
  PHASES.md yet.
- You want to retire a draft cleanly without leaving a closed PR
  in the sub-repo.

Both paths converge on the same end state: a task spec in
`<sub>/tasks/backlog/TASK-NNN-<slug>.md` per claude-kit's
`task-template.md` shape, conforming to
[`governance/task-spec-shape.md`](../.claude/governance/task-spec-shape.md).

---

## Phase proposals

A `proposals/<repo>/PHASES.md` is the orchestrator-side draft of
what the sub-repo's `<sub>/tasks/PHASES.md` will eventually look
like. Same format as `governance/phases-and-tasks.md` "P7 — A
phase's tasks are listed in the phase entry":

```markdown
## Phase N: <noun phrase>

**Outcome:** <one sentence>
**Exit criteria:** <bullets>
**Affects:** <sub-repos>
**Depends on:** <phase IDs or "none">
**Freezes:** <files locked>
**Rollback:** <revert path>
**References:** <URLs, ADRs, prior PRs>

<scope paragraph>

- <task-slug> — <title>
```

Tasks under the phase reference proposed-task slugs (matching
filenames in `proposals/<repo>/backlog/`). At phase promotion,
both the phase entry and all referenced tasks are promoted
together (the natural unit of promotion).

---

## Single-task vs phase promotion

`/promote` accepts either:

- **A task slug** (`/promote refactor-auth-middleware`) — promotes
  one task. If the task has `depends_on:` references to
  unpromoted tasks, the skill warns and asks whether to proceed.
- **A phase ID** (`/promote phase-3`) — promotes the phase entry
  AND all its tasks as one batch. Cross-references inside the
  phase resolve cleanly because everything ships together.

Phase promotion is the recommended path. Single-task promotion is
for exceptions ("I have one urgent task, ship it now").

---

## What you must NOT do

- **Don't bypass `/propose` or `/promote` to write directly into
  `proposals/<repo>/`.** The skills enforce frontmatter shape and
  archive discipline; ad-hoc edits accumulate inconsistencies.
- **Don't delete files from `promoted/` or `retired/`.** They're
  audit history. If a promoted task needs to be re-proposed,
  start a new proposal.
- **Don't promote a proposal whose target sub-repo isn't
  registered or cloned.** Promotion writes into the sub-repo's
  working tree at `repos/<name>/`; the clone must exist.
- **Don't proposal-stage cross-cutting features.** Cross-cutting
  is `/feature`'s territory (orchestrator-level artifact). Per-repo
  phases that *contribute to* a feature live here.

---

## What you must NOT confuse this with

- **Not** `<sub>/tasks/backlog/` — that's the sub-repo's own
  backlog, owned by the sub-kit.
- **Not** `features/` — those are cross-cutting capabilities; this
  is per-repo per-task drafting.
- **Not** `migrations/` — those are coordinated state transitions;
  this is task drafting.
- **Not** `decisions/` — those are ADRs; this is execution
  planning.
- **Not** the `/backlog` skill — that compiles a cross-repo view
  of *existing* sub-repo backlogs. This stages *future* tasks.

---

## See also

- [`_template-task.md`](_template-task.md) — proposal task body
  template
- [`_template-PHASES.md`](_template-PHASES.md) — PHASES.md template
  for first-time creation in a repo's proposals directory
- [`.claude/skills/propose/SKILL.md`](../.claude/skills/propose/SKILL.md)
- [`.claude/skills/promote/SKILL.md`](../.claude/skills/promote/SKILL.md)
- [`.claude/governance/task-spec-shape.md`](../.claude/governance/task-spec-shape.md)
  — the body shape proposals must conform to
- [`.claude/governance/phases-and-tasks.md`](../.claude/governance/phases-and-tasks.md)
  — phase rules that apply to proposed phases just as they do to
  promoted ones
