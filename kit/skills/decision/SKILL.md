---
name: decision
description: Draft a macro-level Architecture Decision Record (ADR) for an architectural call that shapes more than one sub-repo. Stored in decisions/YYYY-MM-DD-short-name.md. Triggered by "/decision", "ADR for X", "let's record why we chose Y", "document the call", "we decided X".
---

# /decision — Macro ADR

Draft an ADR for a real architectural call at the macro level. Per-repo
ADRs live in the sub-repo's own ADR directory; this skill is for
decisions that affect the architecture of the whole company.

Per CLAUDE.md: honest tradeoffs. The point is the reasoning, not the
verdict. No marketing voice.

## When this is the right shape

ADR scale, not commit-message scale. Use this for:

- Choosing a cross-cutting platform / vendor (auth provider,
  observability stack, contract format)
- A non-obvious convention that shapes how all sub-repos work
  (multi-tenancy model, error model)
- A "we considered this and decided not to" that would otherwise be
  lost
- A major direction shift (REST → GraphQL, monolith → services)

Not for:

- Day-to-day per-repo coding decisions — those go in the sub-repo's
  ADR directory
- Anything that fits in a one-line note → `open-questions.md` resolved
  section
- Speculative future work — write the ADR when the decision is being
  made

## Behavior contract

- **Read first.** Read existing ADRs in `decisions/` (chronological,
  most recent first) to understand prior decisions and tone. Read
  `platform-constraints.md`, the relevant `contracts/` files, and the
  related sections of `stack/inventory.md` if applicable.
- **Conversational drafting.** Not a fill-in-the-blank template. Ask
  the user the questions that surface the *real* tradeoff. If they
  can't articulate a rejected alternative, the ADR isn't ready.
- **Don't write a marketing pitch.** "We chose X because Y mattered
  more than Z, even though we lose W" — yes. "We chose X because it's
  great" — no.
- **Don't auto-commit.** Draft the file; user reviews and commits.

## Process

### Step 1 — Surface the decision

Ask the user, one or two at a time:

1. **What's the decision?** One sentence. ("We're adopting OpenAPI as
   the contract source of truth across api / ios / web.")
2. **What's the alternative you rejected?** Most important question.
   If they can't name one, this isn't an architectural call.
3. **Why this one over that one?** What constraint tipped it? Cost,
   fit, ops surface, vendor lock-in, latency, schema match.
4. **What does this cost?** Honest tradeoff. What does the chosen path
   *not* give you that the alternative would have?
5. **What would change your mind later?** Conditions that would
   re-open this decision.
6. **What does this affect?** Which sub-repos, which contract files,
   which migrations does this trigger?

### Step 2 — Generate ID and draft

ID: `YYYY-MM-DD-short-name`. Filename:
`decisions/YYYY-MM-DD-short-name.md`.

Draft using the structure in `decisions/_template.md`. Render the
draft in the response so the user can review before commit.

### Step 3 — Cross-references

If this ADR triggers a migration, mention it. The user can run
`/migration` next to open one. Don't auto-open.

If this ADR updates `contracts/` or `conventions/`, draft those edits
alongside the ADR. Show all changes together.

### Step 4 — Show, confirm, write

Show the ADR draft + any contract/convention edits. Get user
confirmation. Write the files. Don't commit.

## Status field

- **Proposed** — drafted, not yet accepted. Use when filing for
  discussion.
- **Accepted** — decision is in force. Default for ADRs about decisions
  already made.
- **Superseded by [<id>](<id>.md)** — newer ADR replaces this one.
  Don't delete; history is the point.
- **Deprecated** — the decision no longer applies (technology removed
  entirely). Rare.

## Style rules

- **Names for alternatives, not "Option 1 / Option 2."** "OpenAPI",
  "GraphQL", "protobuf" reads better than numbered options.
- **No marketing language.** "Industry-standard", "best-in-class" → cut.
- **Quote constraints from `platform-constraints.md`** when they shape
  the decision.
- **Short.** A good ADR is 1–2 screens. Longer is probably trying to
  be a design doc — different artifact.

## What you must NOT do

- **Don't auto-commit.** ADR + any related edits are reviewed and
  committed by the user.
- **Don't fabricate alternatives.** If only one option was seriously
  considered, say so plainly. Don't invent a strawman.
- **Don't propose ADRs for things that aren't decisions.** A refactor
  isn't an ADR. A bug fix isn't an ADR.

## When NOT to use this skill

- **One-line "we did X" log entry** — append to `open-questions.md`
  resolved section
- **Cross-cutting feature plan** — `/feature`
- **Migration coordination** — `/migration`
- **Per-repo decision** — that's in the sub-repo's own ADR directory

## What "done" looks like

ADR file drafted at `decisions/YYYY-MM-DD-short-name.md`. Any related
contract/convention updates drafted in the same operation. All shown
to the user, uncommitted. The user decides what to commit and when.
