# Proposed phases — `<repo>`

> Orchestrator-side draft of `<repo>/tasks/PHASES.md`. Phases here
> are not yet committed to the sub-repo. Promote via `/promote
> phase-<id>` to push a phase entry + its tasks into
> `<repo>/tasks/PHASES.md` and `<repo>/tasks/backlog/`.
>
> Format matches
> [`governance/phases-and-tasks.md`](../.claude/governance/phases-and-tasks.md)
> "P7 — A phase's tasks are listed in the phase entry." Same
> rules apply to proposed phases as to committed ones — same
> outcome / exit criteria / affects / depends-on / freezes /
> rollback / references discipline.

---

## Phase 1: <noun phrase>

**Outcome:** <one sentence, outside-in>
**Exit criteria:**
- <verifiable bullet>
- <verifiable bullet>

**Affects:** <sub-repos> *(usually just `<repo>`; cross-cutting goes in `/feature`)*
**Depends on:** <phase IDs or "none">
**Freezes:** <files locked during this phase>
**Rollback:** <revert path>
**References:**
- <URL or path>

<2–4 sentence scope paragraph>

- <task-slug-1> — <task title>
- <task-slug-2> — <task title>

---

## Phase 2: <noun phrase>

...

---

<!--
  Add new phases at the bottom. Phase IDs are stable across
  promotion — phase 1 in proposals becomes phase 1 in
  <repo>/tasks/PHASES.md (or appended at the next available
  number if the sub-repo already has phases).

  Tasks listed under each phase are slugs, matching filenames in
  proposals/<repo>/backlog/<slug>.md. At /promote of a phase, all
  listed tasks are promoted together; their slugs become
  TASK-NNN-<slug> in the sub-repo.

  Phases without listed tasks are valid (a phase definition can
  be promoted ahead of its tasks if the user wants the phase
  visible in the sub-repo's PHASES.md before drafting tasks).
-->
