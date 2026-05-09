---
name: brief
description: Generate a synthesized CTO orientation briefing — what changed across sub-repos since last session, open migrations + risks + incidents, stale items, top-of-mind questions. The "Monday morning, what's going on" skill. Triggered by "/brief", "/oncall", "what's going on", "morning briefing", "where do things stand", "give me the rundown", "catch me up", "synthesize state".
---

# /brief — Synthesized CTO orientation

**Output pattern:** [Pattern 23 — Activity timeline](../../output-catalogue.md#23--activity-timeline)
for the changed-since-last-brief section + [Pattern 28 — Stats card grid](../../output-catalogue.md#28--stats-card-grid)
for headline counts at the top.

> ⚠️ **Stub.** Provisional behavior contract — refine on first
> real use. Different from `/status` (snapshot — where things ARE
> right now) in that this *synthesizes the delta* — pulls signals
> for what's CHANGED, AGED, or SURFACED since last session.

The "I just sat down, what do I need to know?" skill. Goal: a
reviewer reads the output in 60 seconds and knows what's actually
different from last session.

## What it pulls

### Across sub-repos (from `repos/<name>/` working trees)

For each registered sub-repo cloned locally (`repos/<name>/` exists):
- `git -C repos/<name> log --since="<last brief or 7 days>"` —
  recent commits.
- `gh pr list -R <owner>/<repo> --state open --search "updated:>$LAST"`
  — PRs with recent activity.
- Last-known HEAD (from `state/last-fetch.json`) vs current HEAD —
  what shipped.

If `state/last-fetch.json` is missing or >24h old, surface the
warning and suggest `/refresh` before continuing — the delta is
unreliable on stale data.

### From the orchestrator

- `state/inbox/<me>.md` — unread messages addressed to me (from
  `.claude/local-config.json` `me` + `inbox_last_read`).
- `migrations/active/` — open migrations + per-repo state + age.
- `risks/open/` — risks aging without movement (>30d).
- `incidents/` — anything not yet `postmortem-filed`.
- `events.md` "Upcoming" — events in the next 14 days.
- `company-notes.md` — entries added in the last 14 days
  (date-headed entries since last brief).
- `open-questions.md` — recently added (last 14d).
- `AUDIT.md` — last 5–10 entries.
- `roadmap.md` "Now" section — top-of-mind work.

## Output shape

```markdown
# CTO brief — YYYY-MM-DD HH:MM

> Headline: <one sentence — "quiet", "incident in flight",
> "migration X stuck, otherwise clean", etc.>

## Changed since last brief

### Sub-repos

| Repo | Commits | Open PRs | HEAD moved |
|---|---|---|---|
| api | 3 | 2 | yes (✅ deployed) |
| ios | 0 | 1 | no |

### Orchestrator artifacts

- _(populate — new migrations, risks, incidents, ADRs, features
  since last brief)_

## Open and aging

### Migrations

- `2026-04-30-auth-token-format-v2` (validating) — ios ✅, web ✅,
  3 validation items left, 7d old

### Risks

- `2026-04-15-postgres-no-replicas` (open, high × medium) — 22d
  without status update

### Incidents

- _(populate or "none open")_

## What I'd look at first

1. _(populate — top 3, in order)_

## Questions surfaced recently

- _(populate from open-questions.md last 14d)_
```

## What you must NOT do

- **Don't write to disk.** This is read-only orientation.
- **Don't auto-filter to "important" without reasoning.** If a
  migration looks fine but is 14d old, surface it — let the user
  judge.
- **Don't fabricate "last brief" timestamp.** Track the timestamp
  somewhere if needed (AUDIT entry on each `/brief` run, or just
  use "since last 7 days" as a default).

## When NOT to use this skill

- Detailed state dump → `/status`.
- Specific question ("what's the state of migration X") → `Read`
  the file directly.
- Onboarding (new collaborator) → `/onboard` (different skill,
  different purpose).
- Compiled cross-repo views — `/roadmap`, `/tasks`, `/backlog` for
  the dimension you care about; `/brief` is the daily-delta layer
  that pulls *changes*, not the full picture.

## What "done" looks like

A rendered briefing in the chat. Nothing written to disk (unless
you decide to log the brief itself to AUDIT — provisional, decide
on first use).
