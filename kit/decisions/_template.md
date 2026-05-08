# ADR — <Short title>

**ID:** YYYY-MM-DD-short-name
**Status:** Proposed | Accepted | Superseded by [YYYY-MM-DD-newer-name](YYYY-MM-DD-newer-name.md) | Deprecated
**Date:** YYYY-MM-DD
**Decider(s):** <names>

## Context

<2–5 sentences. What's the situation that requires a decision? What
constraints exist? What's the question being answered? Cite relevant
files (`stack/inventory.md`, `platform-constraints.md`, etc.) where
the constraints come from.>

## Decision

<1–3 sentences. The actual call. Specific.>

## Alternatives considered

### Option A — <name>

<1–3 sentences on what this would have looked like, and why it was
rejected. Be specific about the tradeoff, not "didn't fit our needs".>

### Option B — <name>

<...>

*(Two minimum. If there wasn't a real alternative, it isn't an
architectural decision — it's a choice. Note the absence honestly
rather than inventing strawmen.)*

## Consequences

**Positive**
- <what we get from this choice that the alternatives wouldn't have>

**Negative / costs**
- <what we lose, what gets harder, what we'll have to revisit>

**Neutral / notable**
- <observations about how the decision shapes future work, without
  classifying as good or bad>

## Revisit triggers

Conditions that, if they occur, mean we should re-open this decision:

- <e.g. "if we add a third client platform">
- <e.g. "if request volume exceeds 10k/s sustained">

## Affects

- Sub-repos: <which ones>
- Contract files updated: <links>
- Migrations triggered: <migration IDs, if any>

## References

- <links to related ADRs, external docs, RFCs, vendor pages>
