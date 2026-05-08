# Decisions (ADRs)

Architecture Decision Records for **macro-level** calls — the choices
that shape the company's stack, not per-repo coding choices.

A per-repo ADR (e.g. "we chose Vue over React for web") lives in that
repo's own `docs/decisions/`. ADRs here are for decisions that affect
the architecture of the whole company.

## When an ADR is the right shape here

- Choosing a cross-cutting platform / vendor (auth provider, cloud,
  observability stack, contract format)
- A non-obvious convention that shapes how all sub-repos work
  (multi-tenancy model, error model, async messaging approach)
- A "we considered this and decided not to" that would otherwise be lost
- A major direction shift (monolith → services, REST → GraphQL,
  per-repo deploys → unified pipeline)

## When NOT

- Day-to-day per-repo decisions — those belong in the sub-repo's own
  ADR directory
- Anything that fits in a one-line note — add to `open-questions.md`
  resolved section
- Speculative future work — write the ADR when the decision is being
  made, not in case

## Naming

`decisions/YYYY-MM-DD-short-name.md`

Example: `2026-05-07-multi-tenant-by-tenant-id.md`

## Lifecycle

ADRs are **append-only**. Once accepted, they don't get edited (except
to mark them superseded). When a later decision changes things, write a
new ADR and reference the old one as superseded.

This makes the file system the audit log of macro decisions.

## Format

See `_template.md`.
