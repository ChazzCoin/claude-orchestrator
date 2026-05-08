---
id: YYYY-MM-DD-short-name
opened: YYYY-MM-DD HH:MM            # UTC
resolved: YYYY-MM-DD HH:MM          # UTC, blank if ongoing
severity: sev1 | sev2 | sev3
status: investigating | mitigated | resolved | postmortem-filed
affects: []                         # sub-repos / domains
related_migrations: []
related_risks: []
related_adrs: []
---

# <Short title>

## Summary

<2–3 sentences. What broke, who was affected, how we noticed.>

## Timeline (UTC)

| Time | Event |
|---|---|
| YYYY-MM-DD HH:MM | _(populate — first signal)_ |
| YYYY-MM-DD HH:MM | _(populate)_ |
| YYYY-MM-DD HH:MM | _(populate — mitigation applied)_ |
| YYYY-MM-DD HH:MM | _(populate — resolution confirmed)_ |

## Impact

<Who was affected, for how long, in what way. Concrete numbers if
you have them — "checkout failed for 8% of users for 47 minutes",
not "users had a bad time".>

## Root cause

<The technical why. Not the "what we did wrong" — the actual
mechanism that produced the failure.>

## Contributing factors

<Things that made this worse than it could have been: missing
monitoring, slow alerting, gap in runbook, single-person knowledge,
contract drift, etc.>

## Why this wasn't caught earlier

<The honest version, not the comfortable one. Gaps in tests,
monitoring, contracts, conventions, or discipline.>

## Resolution

<What we did to fix the immediate issue. Note: this is the
short-term fix. Action items below are the durable response.>

## Action items

| Item | Type | Artifact | Owner | Status |
|---|---|---|---|---|
| _(populate)_ | ADR / risk / migration / sub-repo task / convention | _(link)_ | _(name)_ | open / in-progress / done |

Every action item should resolve to a real artifact within a week
of postmortem filing. Items still "open" past that go to the
monthly review.

## Lessons

<2–4 bullets. What changes about how we work going forward, beyond
the action items. The action items are concrete; the lessons are
the principle.>

## Notes

_(populate — anything unstructured worth keeping: links to PRs,
chats, dashboards, related incidents)_
