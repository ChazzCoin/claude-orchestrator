---
slug: <kebab-case-slug>
title: <descriptive title>
target_repo: <sub-repo-name>
target_phase: <phase-id-from-PHASES.md or "none">
status: backlog        # backlog | retired | promoted
proposed_at: YYYY-MM-DD
proposed_by: <handle>
last_updated: YYYY-MM-DD
depends_on: []          # other proposal slugs OR promoted TASK-NNN IDs
blocks: []
affects: []             # files or modules this would touch on promotion
contract_impact: no     # yes | no — see governance/task-spec-shape.md
references:
  - <url-or-path>
# Promotion fields (populated by /promote at promotion time):
promoted_at:            # YYYY-MM-DD when promoted, or empty
promoted_to:            # TASK-NNN assigned in target sub-repo, or empty
promoted_pr:            # PR URL opened in the sub-repo, or empty
# Retirement fields (populated by /propose retire if abandoned):
retired_at:             # YYYY-MM-DD or empty
retired_reason:         # one line or empty
---

# <Title> *(proposed)*

> Proposed task for `<target_repo>`. Drafted in the orchestrator's
> staging area; not yet committed to the sub-repo. See
> [`proposals/README.md`](README.md) for the lifecycle.
>
> When promoted via `/promote`, this body becomes the body of
> `<target_repo>/tasks/backlog/TASK-NNN-<slug>.md`. The frontmatter
> above is rewritten — proposal-only fields stripped, sub-kit task
> frontmatter substituted per claude-kit's `task-template.md`.

The body sections below are **required** by
[`governance/task-spec-shape.md`](../.claude/governance/task-spec-shape.md).
A proposal that's missing any of these is incomplete and won't
promote cleanly.

## User request

<The human ask, verbatim where possible. Mark *(paraphrased)* if
not verbatim. If the request came from a feature plan or ADR, quote
the relevant paragraph and link the artifact.>

## Why

<One paragraph. The driver. What does the world look like after
this task ships that didn't before?>

## Assumptions

<The doer's restatement of the task in their own words. Confirmed
with the task-filer before code lands. Catches "I read it wrong"
early.>

## Technical breakdown

<Numbered, ordered steps. Each step has an action AND a
verification.>

1. ...
   - **Verify:** ...
2. ...
   - **Verify:** ...

## Acceptance criteria

<Checklist. Each item is a thing a reviewer can run and confirm.
Not "looks correct" — specific, testable.>

- [ ] ...
- [ ] ...

## Out of scope

<What this task is NOT doing. Defends against scope creep
mid-stream.>

- ...

## Contract impact

<Does this touch a public API endpoint, DB schema, event payload,
cross-repo interface, generated type, or shared library?
If yes — link the relevant `bootstrap/contracts/` file or sub-repo
equivalent. If no — write "None.">

## References

<URLs and paths to: official docs, prior PRs, ADRs, migrations,
sub-repo runbooks, orchestrator artifacts. Saved at proposal
creation, not as an afterthought.>

- ...

## Definition of done

Beyond AC — task is "done" when:

- All acceptance criteria are checked.
- Tests for the new behavior exist (per the project's test pairing
  rule from `<sub>/CLAUDE.md`).
- Doc updates land in the same PR if applicable.
- AUDIT entry filed in `<sub>/tasks/AUDIT.md`.
- Advertisement file `<sub>/.claude/active.md` updated.
- If this task is part of an active migration, the
  `<sub>/.claude/active-migrations.md` entry's "This repo's part"
  flips to ✅ for this repo.

---

## Proposal notes *(orchestrator-only — stripped at promotion)*

<Drafting notes, open questions, alternative approaches considered
during proposal iteration. Lives only in the proposal; does not
travel to the sub-repo. Useful for capturing why the proposal is
shaped the way it is.>

- ...
