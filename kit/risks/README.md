# Risks

The risk register. Every surfaced risk gets a file. Open ones live in
`risks/open/`; mitigated ones move to `risks/mitigated/`.

Risks are first-class artifacts at the orchestrator level — alongside
ADRs, features, and migrations. The discipline is: if you can name a
risk concretely, file it. Unnamed risks rot in someone's head.

## When to file a risk vs. another artifact

| You're describing… | Use… |
|---|---|
| "If X happens, we lose Y" — a concrete exposure with severity × likelihood | **risk** |
| "We should change how we do Y" — a deliberate architectural choice | **ADR** (`/decision`) |
| "We need to migrate from X to Y" — coordinated state transition | **migration** (`/migration`) |
| "I'm not sure what to do about X" — open question, no decision yet | **open-questions.md** entry |
| "This already broke" — incident response | **incident** (`/incident`) |

A risk can spawn an ADR (the mitigation decision) and/or a migration
(the work to mitigate). Cross-link via frontmatter `related_adrs`
and `related_migrations`.

## Format

`risks/open/YYYY-MM-DD-short-name.md` — frontmatter + markdown body.
See [`_template.md`](_template.md).

## Severity × likelihood matrix

This is the calibration grid. Don't skip it — "medium / medium" without
reasoning is the noise band.

| Severity \ Likelihood | Low | Medium | High |
|---|---|---|---|
| **Critical** | watch | mitigate | mitigate now |
| **High** | watch | mitigate | mitigate now |
| **Medium** | accept | watch | mitigate |
| **Low** | accept | accept | watch |

- **Critical / High severity** — data loss, security exposure,
  multi-day outage potential, compliance violation.
- **Medium severity** — degraded UX for a subset of users, single-day
  outage potential, compliance friction.
- **Low severity** — annoyance, quick recovery, no user-visible.

Likelihood is calibrated against "in the next 12 months."

## Quarterly risk-surfacing pass

The quarterly review prompts an explicit re-pass: are there risks we
haven't filed yet? What's changed? Empty `risks/open/` is suspicious
— it usually means we haven't been looking.

## Status states

- `open` — surfaced, not yet acted on
- `mitigating` — work in flight (ADR filed, migration opened, or
  per-repo task underway)
- `mitigated` — residual risk acceptable; file moved to `mitigated/`
- `accepted` — consciously holding the risk without mitigation;
  reasoning recorded; reviewed quarterly
