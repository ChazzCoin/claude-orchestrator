---
id: YYYY-MM-DD-<repo>-pr<N>
pr_url: https://github.com/<org>/<repo>/pull/<N>
sub_repo: <name from state/manifest.md>
reviewed: YYYY-MM-DD HH:MM        # UTC
reviewer: orchestrator             # or "orchestrator + <user>" if pair-reviewed
status: approved | conditional | request-changes
pr_status: open | merged | closed
related_migrations: []
related_features: []
related_adrs: []
related_risks: []
related_incidents: []
---

# Orchestrator review — <repo> #<PR>

## What this PR does

<one or two sentences from the PR title + description. Be
factual — not editorial.>

**Files changed:** <count> (<rough breakdown — e.g. "8 source, 2
test, 1 doc">)

## Spec / task alignment

- **Spec source:** <link to `<sub>/tasks/active/TASK-NNN.md`, or
  "PR body — no separate task spec">
- **Stated outcome:** <one line from the spec>
- **Diff appears to do that:** ✅ / ⚠ / ❌
- **Notes:** <where the diff diverges from the spec, if at all;
  or "matches">

## Code reads

- **Cleanly named (variables, functions, files):** ✅ / ⚠ / ❌
- **Organized (logical structure, separation of concerns):** ✅ / ⚠ / ❌
- **Project conventions followed (per `<sub>/CLAUDE.md`):** ✅ / ⚠ / ❌
- **Notes:** <specifics, or "no concerns">

## Macro contract fit

**Naming convention** *(from `conventions/naming.md`):* ✅ / ⚠ / ❌
- <specifics>

**Error handling** *(from `conventions/error-handling.md`):* ✅ / ⚠ / ❌
- <specifics>

**Auth** *(from `conventions/auth.md`):* ✅ / ⚠ / n/a / ❌
- <specifics>

**Contracts touched:** *(from `contracts/{models,endpoints,events}.md`)*
- <contract>: <what changed; does the orchestrator's contract file
  reflect this change? if not, that's drift — call it out>
- <or "none">

**Platform constraints** *(from `platform-constraints.md`):* ✅ / ⚠ / ❌
- <which constraints apply, are they honored?>

**Tech principles** *(from `tech-principles.md`):* ✅ / ⚠ / ❌
- <which principles apply, are they honored or consciously broken?
  if broken — is the reason in the PR body or an ADR?>

## Cross-platform fit

This is the question only the orchestrator can answer.

- **Does this PR change a contract that other sub-repos consume?**
  yes / no
  - If yes: <list affected sub-repos and whether they need a
    related change. Cross-check against `migrations/active/` — is
    there a migration tracking this? If not, opening one may be
    required.>
- **Does this PR depend on a change in another sub-repo that
  hasn't shipped?** yes / no
  - If yes: <which sub-repo, which PR / migration, sequencing>
- **Does this PR break or align with the active migration's
  per-repo plan?** *(if part of one)*
  - <reference migration ID + per-repo plan; does the diff match
    the plan? does this complete the per-repo work?>

## Small gotchas

<bullet list of things noticed — error messages that leak
internal info, hardcoded values that should be config, missing
null checks that the type system can't catch, log lines too
verbose, etc. Don't list line-level lint issues — those are sub-
kit's job. List things that have macro implications: secret
leakage, observability gaps, missing migration coordination, etc.>

- <or "none">

## Risks / incidents reference

- **Did this surface a new risk?** yes / no — <if yes, file via
  `/risk` and reference here>
- **Does this remind me of a past incident?** yes / no — <if yes,
  reference `incidents/<id>.md`. Repeated failure modes are a real
  signal.>

## Action items

- <follow-up artifacts: ADR to file, risk to surface, migration to
  open, contract update needed, convention edit needed>
- <or "none">

## Verdict

**Status:** ✅ approved / ⚠ conditional / ❌ request changes

**Why:** <one sentence summary of the verdict reasoning>

## PR comment to post

*(this is the actual text the orchestrator would post via
`gh pr comment`. Show to the user; post on approval.)*

```markdown
## Orchestrator review (macro fit)

<rendered comment for the PR. Concise — this lives on GitHub, not
in this file. Reference the orchestrator artifact (this file's path
in the orchestrator repo) if the user wants to see the full
review.>
```

## Re-reviews

*(append here if the PR is re-reviewed after updates. Use a
sub-heading per re-review with date.)*

## Notes

<unstructured — anything worth keeping>
