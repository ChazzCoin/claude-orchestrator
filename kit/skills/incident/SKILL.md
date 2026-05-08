---
name: incident
description: Open a new incident, update an in-flight one, draft the postmortem, or close out. Cross-stack incidents only — single-repo postmortems live in the sub-repo's docs/postmortems/. Triggered by "/incident", "log an incident", "we have an outage", "production is down", "draft the postmortem", "close the incident".
---

# /incident — Cross-stack incident skill

> ⚠️ **Stub.** Provisional behavior contract — refine on first
> real use. The discipline (when-here-vs-per-sub-repo, 48-hour
> rule, action-items-must-become-artifacts) is real, see
> [`incidents/README.md`](../../incidents/README.md) and
> [`incidents/_template.md`](../../incidents/_template.md).

Manage `incidents/`. Modes: open, update, postmortem, close.

## Modes

### Open

When something just broke that affects multiple sub-repos OR will
yield cross-cutting lessons:

1. Ask:
   - One-sentence what's broken
   - Severity (sev1 / sev2 / sev3 — see ownership.md scale)
   - Affects (sub-repos / domains)
   - First signal time (when did we notice? UTC)
2. Generate ID: `YYYY-MM-DD-short-name`.
3. Draft from `_template.md` with frontmatter `status:
   investigating`, opened timestamp, severity, affects.
4. Write `incidents/<id>.md`.
5. AUDIT entry: `🔥 Incident opened — <id>. Severity <s>, affects
   <list>. → incidents/<id>.md`.

### Update

Append rows to the Timeline table. Update `status` as it shifts:
`investigating → mitigated → resolved`. AUDIT entries only at
status transitions, not every timeline row.

### Postmortem (within 48h of resolved)

1. Confirm `status: resolved` and `resolved` timestamp set.
2. Walk the user through the body sections: Summary, Impact, Root
   cause, Contributing factors, Why this wasn't caught earlier,
   Resolution, Action items, Lessons.
3. **Action items become artifacts.** For each item, ask: is this
   an ADR, a risk, a migration, a sub-repo task, or a convention
   change? File the artifact (chain to `/decision`, `/risk`,
   `/migration`, etc.) and link from the Action items table.
4. Update frontmatter `status: postmortem-filed`.
5. AUDIT entry: `🩹 Incident resolved — <id>. Postmortem filed
   with N action items. → incidents/<id>.md`.

### Close (housekeeping only)

If a postmortem's action items have all completed, no separate
"closed" status — the incident's done. The followups live in their
own artifacts. The incident file stays in `incidents/` as
historical record.

## Surface rules

- **Open incidents must be surfaced on session start.** If
  `incidents/` has any file with `status: investigating | mitigated`
  (not `resolved` / `postmortem-filed`), the orchestrator surfaces
  it before responding to anything else.
- **Postmortem-due reminders.** If an incident has `status:
  resolved` for more than 48h with no postmortem section filled,
  the orchestrator nudges.

## What you must NOT do

- **Don't file single-repo incidents here.** Lift to here only if
  the lessons span repos. Otherwise per-sub-repo
  `docs/postmortems/`.
- **Don't close without a postmortem.** "We fixed it, moving on"
  is the failure mode this skill exists to prevent.
- **Don't let action items stay un-artifacted.** "We should do X"
  in a postmortem becomes an ADR, risk, or migration before the
  postmortem is filed. Otherwise the postmortem is theater.

## When NOT to use this skill

- Single-repo bug postmortem → sub-kit's `docs/postmortems/`.
- Risk that hasn't materialized → `/risk`.
- Outage that's already fully understood and won't recur → still
  worth a brief postmortem; use the skill but mark the action
  items as "none — preventable by existing controls".

## What "done" looks like

For each phase:

- **Open:** incident file in `incidents/`, AUDIT entry, status
  `investigating`.
- **Update:** timeline appended, status reflects current reality.
- **Postmortem:** body filled, action items linked to artifacts,
  AUDIT entry with 🩹.
