---
name: risk
description: File a new risk in risks/open/, update an existing one, list open risks, or move a risk to mitigated/. Triggered by "/risk", "file a risk", "log a risk", "show open risks", "what risks are we tracking", "this is risky", "we should track this exposure".
---

# /risk — Risk register skill

> ⚠️ **Stub.** Provisional behavior contract — refine on first
> real use. The directory layout, frontmatter, and severity ×
> likelihood matrix are real (see [`risks/README.md`](../../risks/README.md)
> and [`risks/_template.md`](../../risks/_template.md)). The
> mode-by-mode prose below is a sketch.

Manage `risks/open/` and `risks/mitigated/`. Modes: file, update,
list, mitigate, accept.

**Output pattern:** [Pattern 6 — Severity audit](../../output-catalogue.md#6--severity-audit)
for `list` mode (severity-ordered with bars inversely proportional
to severity — criticals are short urgent stubs);
[Pattern 25 — Alert variants](../../output-catalogue.md#25--alert-variants)
for `file` mode (warning variant on confirmation); rendered risk
files follow `risks/_template.md` structure.

## Modes

### File a new risk

1. Ask the user:
   - One-sentence description of the exposure
   - Severity (critical / high / medium / low)
   - Likelihood (high / medium / low) — calibrated against "next
     12 months"
   - Affects (sub-repos from `state/manifest.md` or "stack-wide")
   - Owner (or "unassigned")
2. Generate ID: `YYYY-MM-DD-short-name`.
3. Draft from `kit/risks/_template.md` (synced to
   `risks/_template.md` in this instance). Show the user.
4. On confirmation: write `risks/open/<id>.md`. Append `AUDIT.md`
   entry: `⚠️ Risk surfaced — <id>. Severity <s>, likelihood <l>,
   affects <list>. → risks/open/<id>.md`.

### Update

Append a status update to a risk's "Status updates" section.
Append `AUDIT.md` entry if the update is meaningful (status
change, mitigation chosen, scope shift). Routine "still open" notes
don't go to AUDIT.

### List

Read `risks/open/`, parse frontmatter, render compact table sorted
by severity × likelihood priority (per the matrix in
`risks/README.md`). Flag any open more than 90 days without a
status update.

### Mitigate

1. Verify the chosen mitigation is in place (PR merged, ADR filed,
   migration closed — whatever the file says).
2. Update frontmatter `status: mitigated`.
3. `git mv risks/open/<id>.md risks/mitigated/<id>.md`.
4. AUDIT entry: `🛡 Risk mitigated — <id>. Mitigation: <one-line>.`

### Accept

For risks consciously held without mitigation:

1. Update frontmatter `status: accepted`.
2. Add a "Decision" section noting *why* accepted, *who* signed off,
   *when* to revisit (typically next quarterly review).
3. File stays in `risks/open/` but its status is `accepted`.
4. AUDIT entry: `⚠️ Risk accepted — <id>. Reasoning: <one-line>.
   Re-examine: <date or "quarterly review">.`

## What you must NOT do

- **Don't auto-classify severity or likelihood.** The user calibrates
  these. Asking is the value-add.
- **Don't file risks for things that are already broken.** Those are
  incidents — use `/incident`.
- **Don't file risks for "we should do X someday."** Those are
  open questions — use `open-questions.md`.

## When NOT to use this skill

- Architectural decision with alternatives → `/decision`.
- Active failure → `/incident`.
- Cross-cutting work plan → `/feature` or `/migration`.

## What "done" looks like

A new file in `risks/open/` (or moved to `mitigated/`) and a one-line
AUDIT entry. The user has seen the file and confirmed before write.
