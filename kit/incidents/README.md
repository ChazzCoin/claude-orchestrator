# Incidents

Cross-stack incidents — orchestrator-grade postmortems. Different from
per-repo bug fixes or claude-kit's `docs/postmortems/` (which are
per-codebase).

## When to write one HERE vs. per-sub-repo

| Symptom | Where to write |
|---|---|
| Bug in one repo, fixed in one repo, lessons local | per-sub-repo `docs/postmortems/` |
| Outage spanned multiple sub-repos | **here** |
| Single repo broke but exposed a contract drift / convention gap / cross-cutting failure mode | **here** |
| Single repo broke and the lessons produce action items at the macro level (ADR, risk, convention change) | **here** |
| Vendor outage that affected the stack | **here** |
| Security finding requiring cross-repo response | **here** |

If unsure, write per-sub-repo first. Lift to here only when the
*lessons* span more than one repo.

## 48-hour rule

Every incident gets a postmortem within 48 hours of resolution. Not a
suggestion. The shape is in [`_template.md`](_template.md).

A postmortem with no action items is a story. Real postmortems end
with concrete artifacts:

- ADR filed (the architectural lesson)
- Risk surfaced (`risks/open/`)
- Migration opened (the work to fix it)
- Convention change (`conventions/*.md`)
- Sub-repo task referenced (handoff to per-repo team)

Action items become artifacts. Otherwise the postmortem is theater.

## Format

`incidents/YYYY-MM-DD-short-name.md` — frontmatter + markdown body.

Severity uses the same scale as your on-call rotation if you have one
(sev1 = page everyone, sev2 = page primary, sev3 = next business day).
If you don't have a defined scale, see ownership.md to declare one.

## Cross-linking

Incidents reference and get referenced by:

- **Migrations** — was a migration's missing coordination the
  cause? Frontmatter `related_migrations`.
- **Risks** — did this realize a previously-filed risk? Reference
  it in `related_risks`. If yes, the risk file gets a status update
  pointing at this incident.
- **ADRs** — did this overturn a recorded decision? File a new ADR
  that supersedes the old one.

## After-action review

The first weekly review after an incident closes references it.
The first monthly review after an incident reviews the action items.
The quarterly review evaluates: are we hitting the same kind of
failure repeatedly?
