---
name: promote
description: Promote a proposed task or phase from the orchestrator's staging area at proposals/<repo>/ into the target sub-repo's actual task system, opening a chore/orch-task PR. The promotion path is the only sanctioned route from staging to a sub-repo's tasks/. Triggered by "/promote", "promote <slug>", "promote phase <id> in <repo>", "ship the <slug> proposal", "commit the proposal", "open the task PR for <slug>".
---

# /promote — Push a proposal into its target sub-repo

Move a proposed task or phase from `proposals/<repo>/` into
`<sub-repo>/tasks/`, opening the standard `chore/orch-task-*` PR
in the sub-repo. The natural unit of promotion is a **phase**;
single-task promotion is supported but warns on broken
cross-references.

Background and lifecycle: [`proposals/README.md`](../../../proposals/README.md).
The propose side: [`/propose`](../propose/SKILL.md).

**Output pattern:** [Pattern 23 — Activity timeline](../../output-catalogue.md#23--activity-timeline)
for the per-step promotion progress (read proposal → assign ID →
write sub-repo file → open PR → archive proposal) +
[Pattern 1 — Hero completion card](../../output-catalogue.md#1--hero-completion-card)
on success with the assigned TASK-NNN and PR URL.

---

## Modes

One mode. The skill auto-detects whether the target is a task or
a phase from the user's input.

- **Task promotion** — "/promote `<slug>`", "ship the `<slug>`
  proposal" → promotes a single task.
- **Phase promotion** — "/promote phase-`<id>`", "promote phase 3
  in api" → promotes a phase entry plus all its listed tasks as
  one batch.

If ambiguous, ask which.

---

## Behavior contract

- **The target sub-repo must be cloned.** Promotion writes into
  `repos/<name>/`; if that directory doesn't exist, refuse and tell
  the user to run `bin/setup` to clone first.
- **The proposal must be in `backlog/`** (or for phases, in
  `PHASES.md` and not marked retired). Already-promoted or retired
  proposals are refused.
- **Cross-reference check** — if the proposal's frontmatter
  `depends_on:` references other proposal slugs that haven't been
  promoted yet, warn and ask whether to proceed (the sub-repo
  task's `depends_on:` would point at proposal slugs that don't
  yet exist as sub-repo TASK-NNN IDs).
- **Assign TASK-NNN by reading the sub-repo.** Walk
  `repos/<name>/tasks/{backlog,active,done}/` for the highest
  existing `TASK-NNN-*.md` and use the next integer. Pad to 3
  digits.
- **Open the PR via standard `chore/orch-task-NNN-<slug>` flow**
  per [`sub-projects.md`](../../sub-projects.md) "Coding changes
  — task spec drafted into `<sub>/tasks/backlog/`."
- **Don't auto-merge the PR.** User reviews and merges in the
  sub-repo.
- **Archive the proposal on success** — move from
  `proposals/<repo>/backlog/<slug>.md` to
  `proposals/<repo>/promoted/<slug>.md` with promotion fields
  populated (`promoted_at`, `promoted_to`, `promoted_pr`).

---

## Process: Task promotion

1. **Identify** the target slug. If user said "/promote
   refactor-auth-middleware," locate
   `proposals/<repo>/backlog/<slug>.md` (search across all
   repos if `<repo>` isn't specified; ask if multiple matches).
2. **Read the proposal.** Verify required body sections are
   present per [`governance/task-spec-shape.md`](../../governance/task-spec-shape.md).
   If incomplete, surface what's missing and refuse — incomplete
   proposals don't promote.
3. **Verify sub-repo clone exists** at `repos/<repo>/`. If not,
   tell user to `bin/setup` first.
4. **Pull latest main** in the sub-repo:
   ```sh
   git -C repos/<repo> checkout <default-branch>
   git -C repos/<repo> pull
   ```
5. **Assign TASK-NNN.** Read `repos/<repo>/tasks/{backlog,active,done}/`
   filenames; find the highest `TASK-NNN`; use next integer
   (zero-padded to 3 digits).
6. **Cross-reference check.** Read the proposal's `depends_on:`
   frontmatter. For each referenced slug:
   - If it's a `TASK-NNN` ID → fine (already promoted earlier).
   - If it's a slug AND that slug is in this repo's `promoted/`
     archive → look up the assigned `TASK-NNN`, rewrite the
     reference.
   - If it's a slug AND still in `backlog/` → warn:
     > "This proposal depends on `<other-slug>` which hasn't been
     > promoted yet. The promoted task's `depends_on:` will
     > reference the unpromoted slug. Continue? [yes / no]"
   - If it's neither a known slug nor a `TASK-NNN` → error,
     surface and stop.
7. **Render the sub-repo task spec.** Build
   `repos/<repo>/tasks/backlog/TASK-NNN-<slug>.md`:
   - **Frontmatter:** rewrite per claude-kit's `task-template.md`
     shape:
     - `id: TASK-NNN`
     - `title:` from proposal
     - `phase:` from proposal `target_phase` (if associated)
     - `depends_on:` rewritten per step 6
     - `affects:`, `contract_impact:`, `references:` carried
       through
     - `filed_by: orchestrator`
     - `filed_at:` today's date
     - **Strip:** `slug`, `target_repo`, `target_phase` (now
       redundant), `proposed_at`, `proposed_by`, `last_updated`,
       `promoted_*`, `retired_*`
   - **Body:** carry through all required sections
     verbatim from the proposal. **Strip** the
     `## Proposal notes *(orchestrator-only — stripped at promotion)*`
     section if present.
8. **Branch + write + push + PR:**
   ```sh
   git -C repos/<repo> checkout -b chore/orch-task-NNN-<slug>
   # write the rendered task spec to repos/<repo>/tasks/backlog/TASK-NNN-<slug>.md
   git -C repos/<repo> add tasks/backlog/TASK-NNN-<slug>.md
   git -C repos/<repo> commit -m "orch: TASK-NNN — <title>"
   git -C repos/<repo> push -u origin chore/orch-task-NNN-<slug>
   gh pr create -R <git_remote> --title "orch: TASK-NNN — <title>" --body "<body linking back to proposal + motivating artifact>"
   ```
9. **Also update `<sub>/tasks/ROADMAP.md`** to add the new task
   line under its phase (per claude-kit-reference.md "Updating
   ROADMAP.md when promoting a task spec"). Same PR.
10. **Archive the proposal:**
    - Update proposal frontmatter: `status: promoted`,
      `promoted_at: <today>`, `promoted_to: TASK-NNN`,
      `promoted_pr: <PR URL>`.
    - `git mv` from `proposals/<repo>/backlog/<slug>.md` to
      `proposals/<repo>/promoted/<slug>.md`.
    - If proposal was listed in any phase entry in
      `proposals/<repo>/PHASES.md` task list, leave the entry
      (the phase still contains a reference to the now-promoted
      task; the slug is preserved) but add a marker:
      `<slug> — <title>  *(promoted as TASK-NNN, PR <URL>)*`.

11. **Surface the result:**
    - The TASK-NNN assigned
    - The PR URL
    - The proposal's new location in `promoted/`
    - Reminder that the user merges the PR, the orchestrator does
      not.

---

## Process: Phase promotion

1. **Identify** the phase. If user said "/promote phase-3 in api,"
   locate `## Phase 3` section in `proposals/api/PHASES.md`.
2. **Read the phase entry** + the task slugs listed under it.
3. **Verify sub-repo clone exists** at `repos/<repo>/`.
4. **For each task slug in the phase**, run the task-promotion
   pre-checks (steps 1–6 of task promotion): proposal exists in
   `backlog/`, required sections present, `depends_on:`
   resolved.
   If any task is incomplete or has unresolved dependencies, list
   them and ask whether to:
   - Proceed and skip the problem tasks (phase entry promotes;
     listed tasks that pass promote; flagged tasks stay in
     `backlog/` for the user to fix and re-promote).
   - Abort and let the user fix everything first.
5. **Pull latest main** in the sub-repo.
6. **Assign TASK-NNN to each task** in the order listed in the
   phase. Numbers are sequential starting from the next available.
7. **Determine the sub-repo phase number.** Proposal phase IDs
   (`Phase 1`, `Phase 2`, ...) are local to the proposal; the
   sub-repo's existing `<repo>/tasks/PHASES.md` may already have
   phases with conflicting numbers. Resolve:

   a. Read `repos/<repo>/tasks/PHASES.md` and parse all existing
      phase headings.
   b. If the sub-repo already has phases 1..N, the proposal's
      phases append starting at N+1 by default. Surface the
      mapping to the user:

      > "Mapping proposal phases to sub-repo phase numbers:
      >   - proposals/<repo>/PHASES.md `Phase 1` → sub-repo
      >     `Phase 4` (sub-repo currently at phase 3)
      >   - proposals/<repo>/PHASES.md `Phase 2` → sub-repo
      >     `Phase 5`
      > Proceed? [yes / no]"

   c. **Rewrite each promoted task's `phase:` frontmatter** to
      reflect the assigned sub-repo phase number (not the
      proposal phase number). This is the load-bearing step —
      task spec frontmatter must point at the phase number that
      actually exists in `<repo>/tasks/PHASES.md`, not the one
      from the proposal.
   d. **Rewrite the phase entry's `Depends on:` field**, if it
      references other proposal-phase IDs that have been promoted
      earlier, to the sub-repo phase numbers those map to.
   e. If the sub-repo has placeholder phases (heading but no
      `Outcome:` / `Exit criteria:` body), the user can choose to
      fill those instead of appending. Ask:

      > "Sub-repo `<repo>` has unfilled `Phase <N>` (no body
      > yet). Fill it with this proposed phase, or append at
      > `Phase <M>`?"

   f. Persist the proposal-to-sub-repo phase number mapping in
      the proposal's archived frontmatter at promotion (so the
      audit trail captures which proposal phase mapped where).
8. **Branch + write + push + PR:**
   ```sh
   git -C repos/<repo> checkout -b chore/orch-phase-<id>
   # update tasks/PHASES.md with the new phase entry
   # update tasks/ROADMAP.md with the phase + tasks under it
   # write each task spec to tasks/backlog/TASK-NNN-<slug>.md
   git -C repos/<repo> add tasks/PHASES.md tasks/ROADMAP.md tasks/backlog/
   git -C repos/<repo> commit -m "orch: phase <N> — <name> + <K> tasks"
   git -C repos/<repo> push -u origin chore/orch-phase-<id>
   gh pr create -R <git_remote> --title "orch: phase <N> — <name>" --body "<body linking back to proposal + listing tasks>"
   ```
9. **Archive the proposals:**
   - Each promoted task: archive per task-promotion step 10.
   - The phase entry in `proposals/<repo>/PHASES.md`: keep the
     entry but add `*(promoted <YYYY-MM-DD> as phase <N> in
     <repo>, PR <URL>)*` after the heading.

10. **Surface the result:**
    - The phase number assigned in the sub-repo
    - The TASK-NNN range
    - The PR URL
    - Any tasks skipped due to incompleteness (with reasons)
    - Reminder that the user merges, not the orchestrator.

---

## Style rules

- **Show the diff before pushing.** Render the new sub-repo files
  + the PR title/body so the user sees what's about to be opened.
  Confirm before pushing.
- **PR body must reference back.** Link the proposal file in the
  orchestrator and any motivating artifact (feature plan, ADR,
  migration). The promoted task should be traceable.
- **No emoji.** TASK-NNN, file paths, PR URL.
- **Honest about partial promotions.** If phase promotion skipped
  some tasks, list each skipped task and why.

---

## What you must NOT do

- **Don't auto-merge the PR.** Open it, surface the URL, let the
  user merge.
- **Don't promote a proposal that's incomplete.** Required
  sections per [`governance/task-spec-shape.md`](../../governance/task-spec-shape.md)
  must all be present. Refuse with a list of missing sections.
- **Don't promote without confirming the cross-reference check.**
  If `depends_on:` references unpromoted slugs, the user
  acknowledges the trade-off explicitly.
- **Don't promote a proposal for a sub-repo whose clone is
  missing.** `bin/setup` first; the orchestrator can't render
  the sub-repo files without a working tree.
- **Don't delete the proposal file.** Move to `promoted/` archive
  with full record. Audit trail.
- **Don't rewrite the proposal body** beyond stripping the
  orchestrator-only "Proposal notes" section. The body is the
  contract; promotion preserves it verbatim.
- **Don't bump TASK-NNN past what the sub-repo expects.** Read
  existing tasks; assign sequentially.

---

## When NOT to use this skill

- **Direct-to-sub-repo task drop** without staging — use the
  existing path-3 flow in [`sub-projects.md`](../../sub-projects.md)
  for fully-formed tasks that don't need iteration.
- **Cross-cutting feature** — `/feature` is the orchestrator-level
  artifact; per-repo phases that contribute live in
  `proposals/<repo>/`.
- **Re-promoting** — once a proposal is promoted, the task lives
  in the sub-repo. To change it, edit the sub-repo file directly
  (still through PR per write protocol).
- **Retiring** — that's `/propose retire`, not `/promote`.

---

## What "done" looks like

For task promotion:
- `repos/<repo>/tasks/backlog/TASK-NNN-<slug>.md` rendered per
  claude-kit's task-template.md.
- `repos/<repo>/tasks/ROADMAP.md` updated with the new task line.
- PR `chore/orch-task-NNN-<slug>` opened in the sub-repo with
  reference back to proposal + motivating artifact.
- `proposals/<repo>/promoted/<slug>.md` archived with promotion
  fields populated.
- Phase entry in `proposals/<repo>/PHASES.md` task list updated
  with promotion marker.

For phase promotion:
- `repos/<repo>/tasks/PHASES.md` updated with the new phase entry.
- `repos/<repo>/tasks/ROADMAP.md` updated with phase + tasks.
- One TASK-NNN-<slug>.md per task in `repos/<repo>/tasks/backlog/`.
- One PR `chore/orch-phase-<id>` opened with all of the above.
- Each task proposal archived; phase entry in proposals marked
  promoted.
- Skipped tasks (if any) listed with reasons; they stay in
  `proposals/<repo>/backlog/` for the user to fix.

In both cases: no auto-merge in the sub-repo; user reviews and
merges. No auto-commit in the orchestrator; user reviews and
commits the proposal archive moves.
