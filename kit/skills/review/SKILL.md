---
name: review
description: Run a weekly, monthly, or quarterly review — synthesize state, fill the appropriate template, walk the user through it, file in reviews/. Triggered by "/review", "weekly review", "monthly review", "quarterly review", "kick off review", "let's do the review", "Friday wrap", "month-end review".
---

# /review — Recurring CTO review skill

> ⚠️ **Stub.** Provisional behavior contract — refine on first
> real use. The cadence + template shapes are real, see
> [`reviews/README.md`](../../reviews/README.md) and the three
> `_template-*.md` files.

Run a weekly / monthly / quarterly review. Synthesize state from
the orchestrator, walk the user through the right template, file
the result.

**Output pattern:** [Pattern 28 — Stats card grid](../../output-catalogue.md#28--stats-card-grid)
for headline metrics + [Pattern 23 — Activity timeline](../../output-catalogue.md#23--activity-timeline)
for the period's chronology. The filed review file follows
`reviews/_template-*.md` markdown structure (not catalogued — those
are durable artifacts, not terminal output).

## Modes — detect from input

- "weekly", "Friday wrap", "Monday" → weekly mode
- "monthly", "month-end", "this month" → monthly mode
- "quarterly", "quarter end", "Q[1-4]" → quarterly mode
- ambiguous → ask which

## Common process

Whichever mode:

1. **Compute the period.** Weekly: ISO week (`date +%G-W%V`).
   Monthly: `YYYY-MM` of the period being reviewed (last full
   month, not in-progress). Quarterly: `YYYY-QN` of last full
   quarter.
2. **Check for prior review** in the same period — refuse to
   double-file. If one exists, offer to read + amend instead.
3. **Pull source state** from the orchestrator (see per-mode
   sections below).
4. **Fill the template** with the user's voice — interview-driven
   for the qualitative parts, automated for the quantitative.
5. **Show the draft.** Get edits.
6. **File** at `reviews/<mode>/<period>.md`.
7. **AUDIT entry** with 🎯: `🎯 <Mode> review filed — <period>.
   Headline: <one-line>. Artifacts: <list>. → reviews/...`.

## Mode: weekly

**Source pull:**
- `git -C repos/<name> log --since="<week start>"` for each cloned
  sub-repo — what shipped per repo. (If `repos/<name>/` doesn't
  exist, fall back to `gh api repos/<owner>/<repo>/commits` for
  the same window.)
- `migrations/active/` — for each, age + per-repo state.
- `risks/open/` — flag anything updated this week.
- `incidents/` — flag anything opened or resolved this week.
- `events.md` "Upcoming" + "Past" — events in the past week (what
  happened) and next two weeks (what's coming).
- `state/inbox/*.md` — surface inboxes with new entries this week.
- `company-notes.md` — entries added this week.
- `open-questions.md` — what was added.

**Walk the user through** the weekly template's qualitative
sections: Headline, Stuck or stale, At risk this week, Next week's
top 3, Inbox follow-ups.

## Mode: monthly

**Source pull:**
- All weekly reviews from the period (in
  `reviews/weekly/YYYY-WW*.md` matching the month) — roll up.
- `migrations/closed/` — what closed this month.
- `decisions/` — ADRs filed this month.
- `cost-tracking.md` — current month vs last.
- `slos.md` — current posture.
- `incidents/` from the month.
- `vendors.md` — anything renewed / changed.
- `events.md` "Past" — events that occurred this month (notable
  conferences, demos, launches).
- `company-notes.md` — month's notes (signals to amplify or move
  to artifacts).

**Walk through** the monthly template — heavier on rolled-up data,
lighter on free-form.

## Mode: quarterly

**Source pull:**
- All monthly reviews from the period.
- `roadmap.md` — diff the start-of-quarter snapshot against now.
- `tech-vision.md`, `tech-principles.md` — re-read both.
- `company-profile.md` "Strategic direction" — re-read; flag if
  drifted from actual quarter outcomes.
- All `risks/open/` — full review pass.
- `compliance.md`, `security-posture.md`, `vendors.md` — each
  reviewed.
- Compiled views from the period: run `/roadmap`, `/tasks`,
  `/backlog` and review them as part of strategic recalibration.

**Walk through** the quarterly template's strategic sections:
Roadmap reality check, Principles re-examination, Vision
check-in, Big bet for next quarter, Risk-surfacing pass *(this is
mandatory — don't skip)*.

## What you must NOT do

- **Don't auto-file without user review.** The whole point is the
  human's calibration on the qualitative sections.
- **Don't merge two periods into one review.** A skipped week is a
  short "skipped — reason: …" entry, not a doubled-up review.
- **Don't fabricate metrics.** If `cost-tracking.md` or `slos.md`
  isn't being maintained, the review template's relevant sections
  say "not yet instrumented" — not made-up numbers.

## When NOT to use this skill

- Daily orientation / status check → `/status` or `/brief`.
- Strategic recalibration outside the cadence → that's a
  `/decision` (ADR), not a review.
- Postmortem after an incident → `/incident` postmortem mode.

## What "done" looks like

A new file in `reviews/<mode>/<period>.md`, walked-through with
the user, with action items spawned where applicable (ADRs, risks,
migrations, roadmap edits filed before the review is "done"). AUDIT
entry with 🎯.
