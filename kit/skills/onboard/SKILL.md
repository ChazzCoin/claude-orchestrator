---
name: onboard
description: Synthesize the orchestrator's current macro state into a guided walkthrough — what this company's stack is, how it's built, what's in flight, what's been decided. For a new collaborator, or future-you returning after a gap. Triggered by "/onboard", "walk me through this", "I'm new", "where am I", "remind me what's going on".
---

# /onboard — Synthesize orchestrator state

Render a guided walkthrough of the current macro state. Not a doc dump
— a sequence: what to know first, what's in flight now, what's been
decided.

For a new collaborator landing in this orchestrator instance cold, or
for future-you returning after weeks away.

**Output pattern:** [Pattern 30 — Multi-step wizard](../../output-catalogue.md#30--multi-step-wizard)
shape (numbered sections as steps the reader walks through) +
[Pattern 33 — Command reference](../../output-catalogue.md#33--command-reference)
for the "how to operate" section.

## Behavior contract

- **Read the canonical sources, in order:**
  1. `README.md` — what this orchestrator instance is for
  2. `CLAUDE.md` — the working contract
  3. `company-profile.md` — mission, products, leadership, mottos,
     strategic direction
  4. `stack/inventory.md` — what we have
  5. `stack/infra-map.md` — where it runs
  6. `platform-constraints.md` — what bounds design
  7. `contracts/` — current contract surface (markdown spec)
  8. `conventions/` — naming, errors, auth
  9. `migrations/active/` — what's in flight cross-repo
  10. `decisions/` — recent ADRs (last 5 by date)
  11. `roadmap.md` — current direction
  12. `events.md` — upcoming events (conferences, demos, launches)
  13. `open-questions.md` — what's parked

  Any may be partial; render the section without it rather than
  guessing.

- **Prerequisite check.** If `repos/` is empty (no sub-repo
  checkouts), recommend `bin/setup` before proceeding — the
  walkthrough's "in flight" section needs sub-repo state to be
  meaningful.

- **Render as a guided walkthrough, not a digest.** Group by "what
  this is" → "the stack" → "the contracts" → "what's in flight" → "what's
  been decided" → "open questions" → "where to look next."

- **Cite, don't paraphrase.** When a file has the answer, link to it
  rather than restating.

- **Surface gaps honestly.** If a section is unpopulated, say so —
  don't fill from imagination.

## Output structure

```markdown
# 👋 Orchestrator overview — <company>

> **What this is.** <one sentence pulled from README.md or company-profile.md>
> **Where things are.** <one sentence — N sub-repos, M active migrations,
> K open questions, last decision YYYY-MM-DD>

---

## 1. The company
*From `company-profile.md`.*

- **Mission:** <one line from "Mission" section>
- **Products:** <product names, one line each if more than one>
- **Strategic direction:** <one line from "Strategic direction (current)">
- **Mottos:** <one line, max 2 mottos>

For depth: [`company-profile.md`](company-profile.md).

---

## 2. The stack
*From `stack/inventory.md`.*

- **api** — <one line>
- **ios** — <one line>
- **web** — <one line>
- **devops** — <one line>

For depth: [`stack/inventory.md`](stack/inventory.md),
[`stack/infra-map.md`](stack/infra-map.md).

---

## 3. The constraints that bound design
*From `platform-constraints.md`. The 3–5 highest-leverage rules.*

- <bullet>
- <bullet>

For depth: [`platform-constraints.md`](platform-constraints.md).

---

## 4. The contracts
*From `contracts/`.*

- **Models:** <count of entities, one-line summary>
- **Endpoints:** <count, surface area>
- **Events:** <count, or "none today">

For depth: [`contracts/`](contracts/).

---

## 5. In flight
*From `migrations/active/` and `roadmap.md` "Now".*

**Active migrations (<count>):**
- <id> — <one line>

**Currently steering toward:**
- <roadmap "Now" item>

For depth: `/status`, then individual migration files.

---

## 6. Coming up
*From `events.md` "Upcoming" — next 14 days only.*

- <YYYY-MM-DD · title> *(or "no events scheduled")*

For depth: [`events.md`](events.md).

---

## 7. Recent decisions
*From `decisions/`, last 5 by date.*

- **<YYYY-MM-DD-name>** — <one line on the call>

For depth: [`decisions/`](decisions/).

---

## 8. Open questions
*From `open-questions.md`. Active items count + any tagged blocking.*

- <blocking items, with reason>
- *(N more deferred — see [`open-questions.md`](open-questions.md))*

---

## 9. How to operate
*Brief.*

**Daily:**
- `/status` — current macro picture (from cached state)
- `/refresh` — fetch latest from all sub-repos
- `/brief` — what's changed since last session
- `/inbox` — read / send addressed messages

**Compilers:**
- `/roadmap` — phase view across all sub-repos
- `/tasks` — active task view across all sub-repos
- `/backlog` — queued backlog across all sub-repos
- `/sync-check` — drift between sub-repos and contracts

**Authoring (write artifacts):**
- `/decision` — file an ADR
- `/feature` — author a cross-cutting feature plan
- `/migration` — open / update / close cross-repo migration
- `/risk` — file or update a risk
- `/incident` — log a cross-stack incident, draft postmortem
- `/review` — run weekly / monthly / quarterly review

**Setup / maintenance:**
- `/register` — add a new sub-repo
- `bin/setup` — clone all registered sub-repos (collaborator onboarding)
- `/sync` — pull updates from claude-orchestrator kit
- `/audit` — populate / refresh stack, contracts, conventions

The full skill index lives in [`.claude/skills/`](.claude/skills/).

---

## ⚠️ Gaps I noticed *(if any)*

*Render this only if files were genuinely incomplete.*

- <gap with file path>
- ...

To address: run `/audit` to resume the audit, or edit files directly.
```

## Style rules

- **Imperative voice in the operate section.** "Read X. Run Y."
- **Markdown links to source files.** Reader clicks through.
- **Section numbers (1–7).** They're a sequence.
- **No padding.** Sections without content get omitted.

## What you must NOT do

- **Don't extrapolate beyond what's documented.** If a section is
  unpopulated, note the gap rather than filling from training data.
- **Don't recommend changes.** That's not this skill's job.
- **Don't quote contract content verbatim.** Summarize, point at the
  file.
- **Don't run `/sync-check` automatically.** State on disk is what
  this skill renders.

## When NOT to use this skill

- **Already oriented** → `/status` for today's picture.
- **Specific question** → ask it directly; don't run the full
  walkthrough.
- **Setting up the orchestrator from scratch (no populated files)** →
  `/audit` first; this skill needs source content.

## What "done" looks like

A single rendered walkthrough. The reader can read it top-to-bottom
in ~5 minutes and know what this orchestrator is for, what stack it
covers, what's in flight, what's been decided, and what to do next.
No file edits, no commits.
