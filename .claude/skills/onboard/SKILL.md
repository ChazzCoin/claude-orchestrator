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

## Behavior contract

- **Read the canonical sources, in order:**
  1. `README.md` — what this orchestrator instance is for
  2. `CLAUDE.md` — the working contract
  3. `stack/inventory.md` — what we have
  4. `stack/infra-map.md` — where it runs
  5. `platform-constraints.md` — what bounds design
  6. `contracts/` — current contract surface (markdown spec)
  7. `conventions/` — naming, errors, auth
  8. `migrations/active/` — what's in flight cross-repo
  9. `decisions/` — recent ADRs (last 5 by date)
  10. `roadmap.md` — current direction
  11. `open-questions.md` — what's parked

  Any may be partial; render the section without it rather than
  guessing.

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

> **What this is.** <one sentence pulled from README.md>
> **Where things are.** <one sentence — N sub-repos, M active migrations,
> K open questions, last decision YYYY-MM-DD>

---

## 1. The stack
*From `stack/inventory.md`.*

- **api** — <one line>
- **ios** — <one line>
- **web** — <one line>
- **devops** — <one line>

For depth: [`stack/inventory.md`](stack/inventory.md),
[`stack/infra-map.md`](stack/infra-map.md).

---

## 2. The constraints that bound design
*From `platform-constraints.md`. The 3–5 highest-leverage rules.*

- <bullet>
- <bullet>

For depth: [`platform-constraints.md`](platform-constraints.md).

---

## 3. The contracts
*From `contracts/`.*

- **Models:** <count of entities, one-line summary>
- **Endpoints:** <count, surface area>
- **Events:** <count, or "none today">

For depth: [`contracts/`](contracts/).

---

## 4. In flight
*From `migrations/active/` and `roadmap.md` "Now".*

**Active migrations (<count>):**
- <id> — <one line>

**Currently steering toward:**
- <roadmap "Now" item>

For depth: `/status`, then individual migration files.

---

## 5. Recent decisions
*From `decisions/`, last 5 by date.*

- **<YYYY-MM-DD-name>** — <one line on the call>

For depth: [`decisions/`](decisions/).

---

## 6. Open questions
*From `open-questions.md`. Active items count + any tagged blocking.*

- <blocking items, with reason>
- *(N more deferred — see [`open-questions.md`](open-questions.md))*

---

## 7. How to operate
*Brief.*

- `/status` — daily overview
- `/sync-check` — refresh sub-repo state
- `/migration` — open / update / close cross-repo changes
- `/decision` — file an ADR
- `/feature` — author a cross-cutting feature plan

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
