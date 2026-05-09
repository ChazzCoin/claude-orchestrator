---
name: propose
description: Draft, edit, retire, or list proposed tasks and phases in the orchestrator's per-repo staging area at proposals/<repo>/. Drafts live there until promoted to the sub-repo via /promote — letting the CTO plan multiple repos in parallel without opening PRs in those repos prematurely. Triggered by "/propose", "draft a task for <repo>", "propose a phase for <repo>", "stage a task before committing", "edit the <slug> proposal", "retire the <slug> proposal", "what have I proposed for <repo>", "show proposed tasks", "list proposals".
---

# /propose — Draft & manage proposed tasks and phases

The orchestrator's per-repo staging area for tasks and phases that
aren't ready to commit to the sub-repo yet. Use this when you want
to plan, draft, and iterate before opening any PRs in sub-repos.

When ready to commit, use [`/promote`](../promote/SKILL.md) to push
a proposal into its target sub-repo's actual task system.

Background and lifecycle: [`proposals/README.md`](../../../proposals/README.md).

**Output pattern:** [Pattern 4 — Sprint task board](../../output-catalogue.md#4--sprint-task-board)
for `list` mode (per-repo grouping with proposal counts and status
symbols) + [Pattern 1 — Hero completion card](../../output-catalogue.md#1--hero-completion-card)
on `new` / `update` / `retire` confirmation. Drafted proposal files
follow [`proposals/_template-task.md`](../../../proposals/_template-task.md)
and [`proposals/_template-PHASES.md`](../../../proposals/_template-PHASES.md).

---

## Modes

Detect from user input. Object type is `task` (default) or `phase`.

- **New** — "/propose new", "draft a task for `<repo>`",
  "propose a phase for `<repo>`" → write a new proposal.
- **Update** — "/propose update `<slug>`", "edit the
  `<slug>` proposal", "revise phase `<id>` in
  `<repo>`" → open the proposal for revision.
- **Retire** — "/propose retire `<slug>`", "abandon the
  `<slug>` proposal", "drop phase `<id>` proposal" → move to
  `retired/` with reason.
- **List** — "/propose list", "what have I proposed for
  `<repo>`", "show all proposals" → enumerate proposals.

If ambiguous, ask which.

---

## Behavior contract

- **The proposal target sub-repo must be registered.** If
  `state/manifest.md` doesn't list the repo, refuse and tell the
  user to `/register` first.
- **Don't write into `repos/<name>/`.** Proposals live in the
  orchestrator at `proposals/<repo>/`. Writing to the sub-repo is
  `/promote`'s job.
- **Required body sections** (per
  [`governance/task-spec-shape.md`](../../governance/task-spec-shape.md))
  must all be present in any task proposal. The Q&A flow walks the
  user through each. Skipping is allowed but produces a draft
  flagged "incomplete" that won't `/promote` cleanly.
- **Slugs are stable identifiers.** Once chosen, a proposal's slug
  doesn't change. Filename = `<slug>.md`. Slug becomes part of the
  promoted filename `TASK-NNN-<slug>.md`.
- **Phase IDs use small integers** matching the order in
  `proposals/<repo>/PHASES.md`. They mean "phase 1 in this repo's
  proposals" — not the same as the eventual phase number in
  `<sub>/tasks/PHASES.md` (which depends on the sub-repo's
  existing phases).
- **Don't auto-commit.** All writes leave files dirty for the user.

---

## Mode: New

### Sub-mode: New task

1. **Identify the target sub-repo.** Ask if not stated. Verify it's
   in `state/manifest.md`.
2. **Identify the slug.** Either explicit ("the slug should be
   `refactor-auth-middleware`") or derived from the user's
   description (kebab-case, ≤40 chars). Verify no collision with
   existing proposals in `proposals/<repo>/backlog/`,
   `promoted/`, or `retired/`.
3. **Q&A flow** — walk the required sections of
   [`governance/task-spec-shape.md`](../../governance/task-spec-shape.md):
   - User request (or paste from upstream artifact)
   - Why
   - Assumptions (the doer's reading restated)
   - Technical breakdown (numbered + verifiable)
   - Acceptance criteria (testable)
   - Out of scope
   - Contract impact
   - References (URLs, prior PRs, ADRs)
   - Definition of done (auto-populated default; user can adjust)
4. **Phase association.** Ask which phase this proposal is for. If
   `proposals/<repo>/PHASES.md` doesn't exist or has no matching
   phase, offer to create the phase as part of this flow.
5. **Render the draft** using
   [`proposals/_template-task.md`](../../../proposals/_template-task.md).
   Show to user. Confirm.
6. **Write** to `proposals/<repo>/backlog/<slug>.md`, creating the
   `proposals/<repo>/{backlog,promoted,retired}/` directories if
   absent.
7. **If a phase was associated**, add the task slug to that phase's
   task list in `proposals/<repo>/PHASES.md`.
8. **Surface the next move:** `/propose update <slug>` to revise,
   `/promote <slug>` when ready.

### Sub-mode: New phase

1. **Identify the target sub-repo.** Same as task flow.
2. **Q&A flow** — walk the phase-shape rules from
   [`governance/phases-and-tasks.md`](../../governance/phases-and-tasks.md):
   - Outcome (one sentence, outside-in)
   - Exit criteria (verifiable bullets)
   - Affects (usually just this repo)
   - Depends on (other phases or "none")
   - Freezes (files locked)
   - Rollback path
   - References
   - Scope paragraph (2–4 sentences)
3. **Identify the phase ID.** Default = next integer after the last
   phase in `proposals/<repo>/PHASES.md`. User can override.
4. **Render the draft** using the entry shape in
   [`proposals/_template-PHASES.md`](../../../proposals/_template-PHASES.md).
   Show to user. Confirm.
5. **Write** to `proposals/<repo>/PHASES.md` (creating the file
   from template if absent). Append the new phase entry at the
   bottom.
6. **Surface the next move:** propose the tasks under this phase
   with `/propose new` or `/promote phase-<id>` once tasks are
   ready.

---

## Mode: Update

1. **Identify** target slug or phase ID + repo.
2. **Locate the file.** Tasks: `proposals/<repo>/backlog/<slug>.md`.
   Phases: `proposals/<repo>/PHASES.md` `## Phase <id>` section. If
   absent, surface and stop.
3. **Show the current content.** Ask which sections to revise.
4. **Apply revisions** — re-prompt for the sections the user wants
   to change. Update `last_updated` frontmatter (tasks) or note
   the revision in a `<!-- revised <YYYY-MM-DD>: <reason> -->`
   comment (phases).
5. **Show the updated draft.** Confirm.
6. **Write.** Don't auto-commit.

Update is the iteration mechanism. Use it freely; that's the whole
point of staging.

---

## Mode: Retire

1. **Identify** slug or phase ID + repo.
2. **Locate the file** in `backlog/` (tasks) or `PHASES.md`
   (phases).
3. **Read it.** Show the current content.
4. **Ask for a one-line retire reason** (helpful future audit).
5. **Confirm** before moving:

   > "Retire `<slug>` from `<repo>` proposals? Moves to
   > `proposals/<repo>/retired/`. The file is preserved; you can
   > re-propose later by copying back. [yes / no]"

6. **For tasks:**
   - Update frontmatter: `status: retired`, `retired_at: <today>`,
     `retired_reason: <one line>`.
   - `git mv` the file to `proposals/<repo>/retired/<slug>.md`.
   - If the task was listed under any phase in `PHASES.md`, remove
     it from the phase's task list.

7. **For phases:**
   - Mark the phase entry in `PHASES.md` with `*(retired
     <YYYY-MM-DD>: <reason>)*` after the heading.
   - If any tasks were under this phase, ask the user whether to
     also retire them, leave them under no phase, or move them to
     a different phase.

8. **Surface what was done.**

---

## Mode: List

### Process

1. Determine scope. Either:
   - Specific repo: `/propose list <repo>` → list that repo's
     proposals.
   - All repos (default): walk every `proposals/<repo>/` directory.
2. For each in-scope repo:
   - Count proposals in `backlog/` (active drafts), `promoted/`
     (archive), `retired/` (archive).
   - Read `PHASES.md` if present; count phases (proposed and
     retired).
3. Render compact:

   ```
   proposals:

     api      backlog: 3  · phases proposed: 2  (1 retired)
       - refactor-auth-middleware  *(updated 2026-05-08)*
       - tenant-id-on-users
       - audit-log-cleanup
     ios      backlog: 1  · phases proposed: 0
       - tenant-switcher-ui
     web      *(no proposals yet)*
     devops   *(no proposals yet)*

   archive:
     api      promoted: 5  · retired: 1
     ios      promoted: 2  · retired: 0

   next moves: /propose new, /propose update <slug>, /promote <slug>
   ```

4. If `--full` or "with details," render each proposal with title +
   target phase + last_updated.

### Style

- Group by repo.
- Backlog before archive.
- Don't dump full body; that's `Show` (TODO: future mode if
  needed).

---

## Style rules

- **Keep proposals under one screen.** A proposal that's growing
  past 2 screens probably wants splitting into multiple proposals
  (T1: one task = one reviewable diff).
- **Don't pad.** A proposal is a draft; padding it with hedging
  language doesn't help iteration.
- **Capture references at proposal time.** Same discipline as
  task-spec-shape.md — a proposal without grounded references is
  a proposal built on assumptions.
- **No emoji decoration** beyond ⚠ for incomplete proposals
  (missing required sections) and 🗃 for retired ones in `list`
  mode.

---

## What you must NOT do

- **Don't write to `repos/<name>/`.** That's `/promote`'s
  responsibility.
- **Don't change a slug after creation.** Filename, frontmatter,
  PHASES.md references would all need updating; too easy to leave
  inconsistencies. Retire and re-propose if the slug is wrong.
- **Don't promote from this skill.** `/promote` is the only path
  from staging to sub-repo.
- **Don't create a proposal for a sub-repo not in the manifest.**
  Refuse with "register the sub-repo first."
- **Don't delete files in `retired/` or `promoted/`.** Audit trail.
- **Don't auto-commit.**

---

## When NOT to use this skill

- **Cross-cutting feature plan** — `/feature` (orchestrator-level
  artifact); per-repo phases that *contribute* to a feature live
  here.
- **Direct-to-sub-repo task drop** — use the existing path-3 flow
  in [`sub-projects.md`](../../sub-projects.md) when the task is
  fully formed and you don't need iteration.
- **Cross-repo aggregated view of existing tasks** — `/tasks` or
  `/backlog`. This skill is for *future* tasks not yet in any
  sub-repo.
- **Architectural decision** — `/decision`.
- **Reading sub-repo state** — `/sync-check`, `/status`.

---

## What "done" looks like

For each mode:

- **New (task):** `proposals/<repo>/backlog/<slug>.md` written
  per template; phase association added to PHASES.md if any.
  User shown the draft and the next-move options.
- **New (phase):** `proposals/<repo>/PHASES.md` updated with new
  phase entry. User shown the entry and the next-move options.
- **Update:** target file edited in place; `last_updated`
  refreshed. User shown the diff.
- **Retire:** target moved to `retired/` (tasks) or marked retired
  (phases) with reason. PHASES.md task lists cleaned up.
- **List:** rendered scoped enumeration with counts and next-move
  pointers.

In all cases: no auto-commit. The user commits.
