---
id: YYYY-MM-DD-short-name
surfaced: YYYY-MM-DD
severity: critical | high | medium | low
likelihood: high | medium | low
status: open       # open | mitigating | mitigated | accepted
affects: []        # sub-repo names from state/manifest.md, or "stack-wide"
owner:             # person or "unassigned"
related_adrs: []
related_migrations: []
related_incidents: []
---

# <Short title>

## What's at risk

<2–4 sentences. The exposure in concrete terms. Be specific —
"could lose 12h of audit data if Postgres dies between hourly
snapshots", not "data loss possible".>

## Why now

<What surfaced this. Incident? Audit finding? Discovery during
work? External event (CVE, vendor outage, regulatory change)?>

## Severity × likelihood reasoning

<Calibrated, not vibes. Why this severity (what's the
worst-realistic outcome?). Why this likelihood (what's the trigger
event, how likely in the next 12 months?). Reference data if you
have it.>

## Mitigation options

- **<option 1>** — cost: <effort estimate>; residual risk: <what's
  still exposed after this option>; tradeoffs: <what we give up>
- **<option 2>** — …
- **<accept>** — keep this risk; reasoning: …

## Decision

<What we chose, when, by whom. Or "deferred — see
open-questions.md entry YYYY-MM-DD-…". Link the ADR if there is
one.>

## Status updates

- **YYYY-MM-DD** — _(populate as work happens)_
