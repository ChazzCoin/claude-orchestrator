---
name: audit
description: Interview the user to populate or refresh the orchestrator's macro state — stack inventory, infra map, contracts, conventions, platform constraints. The v1 entry point for a fresh orchestrator instance. Triggered by "/audit", "let's do the audit", "interview me", "populate the orchestrator", "wrangle the stack", "what do we have".
---

# /audit — Macro audit by interview

The orchestrator only works if its files reflect reality. This skill is
how reality gets onto disk.

This is **interview-driven**, not autonomous discovery. The user is the
ground truth for the macro picture (some of which lives only in their
head). The orchestrator's job is to ask the right questions in the
right order, write answers to the right files, and surface what's still
missing.

**Output pattern:** [Pattern 30 — Multi-step wizard](../../output-catalogue.md#30--multi-step-wizard)
for the phase progression (current phase highlighted, others queued
or done) + [Pattern 26 — Empty state](../../output-catalogue.md#26--empty-state)
when a section can't be filled (capture as open question, move on).

Per CLAUDE.md: blunt resonant honesty. If the user can't answer, they
can't answer — note it as an open question and move on. Don't fill in
guesses.

## Behavior contract

- **One area at a time.** Don't dump the whole question list at once.
  Move through phases sequentially; finish a phase, write its file,
  then start the next.
- **Confirm before writing.** When a phase is complete, show the user
  the structured content you're about to commit to its file. They edit
  or accept.
- **Resume across sessions.** The audit will not finish in one sitting.
  At end of session, summarize what's covered, what's left, and where
  to pick up. Each subsequent `/audit` reads the existing files first
  and asks "where did we leave off?".
- **Don't fabricate.** "I don't know" is a valid answer. Capture it as
  an entry in `open-questions.md`.
- **Cite where claims come from.** If the user says "the API is on
  Azure Container Apps in eastus", that goes in `stack/infra-map.md`
  with the source noted in a comment if it's worth tracking.

## Phases (in order)

### Phase 1 — Two-company shape *(skip if single company)*

If the user has only one stack, skip. Otherwise, confirm: "this
orchestrator instance is for which company?" — name and one-line
purpose. Write to `README.md` if not already populated.

### Phase 2 — Stack inventory

Goal: populate `stack/inventory.md`.

Ask:

1. "List every sub-repo in this company's stack." Get names.
2. For each:
   - One-sentence purpose
   - Stack (language, framework, runtime version)
   - Where it runs (deploy target)
   - What it exposes (UI, endpoints, infra, events)
   - What it consumes (which other components, by name)
   - Production state (production / beta / in-development)
   - Any gotchas worth recording

Write each repo as a section in `stack/inventory.md` using the format
the file already documents. After all repos, ask about out-of-stack
dependencies (auth providers, third-party APIs, payment, analytics).

### Phase 3 — Infra map

Goal: populate `stack/infra-map.md`.

Ask in order:

1. Cloud — provider, accounts, regions
2. Compute — clusters / function apps / container apps, per env
3. Networking — ingress, DNS, public vs private
4. Data plane — DBs, caches, object storage, queues
5. Identity & secrets — workload identity, secret store
6. CI/CD — pipelines, deploy authority, rollback
7. Observability — metrics, logs, traces, alerts

If the devops sub-kit is registered, mention that some of this can be
auto-populated by the devops kit and ask whether to defer those
sections.

### Phase 4 — Platform constraints

Goal: populate `platform-constraints.md`.

This is the highest-leverage section. The constraints recorded here
will inform every contract design later.

Ask, for each category in `platform-constraints.md`:

- "What's the actual limit / behavior here?"
- "Has this caused a design decision before?"
- "What would happen if we ignored it?"

When a constraint is concrete enough, draft a "Design rule that falls
out" entry in the same file (e.g. "ingress timeout 30s → no synchronous
endpoint > 25s budget").

### Phase 5 — Contracts (current state)

Goal: populate `contracts/models.md`, `contracts/endpoints.md`,
`contracts/events.md` with **what currently exists**, not what should
exist.

Ask:

1. "What are the core entities in your domain?" — get names. For each:
   purpose, owner repo, key fields, how it gets created/mutated/deleted.
2. "What endpoints exist today?" — get the surface. For each: path,
   method, owner, consumers, auth requirement.
3. "What async messaging exists, if any?" — events, queues, topics. If
   none, document that explicitly.

Many gaps will surface here. That's the point. File each gap in
`open-questions.md` with the question that needs to be answered to fill
the gap.

### Phase 6 — Conventions

Goal: populate `conventions/naming.md`,
`conventions/error-handling.md`, `conventions/auth.md`.

Ask:

- "What's the naming convention on the wire? In each platform? Where
  does transformation happen?"
- "What's the error shape? Status code mapping? Client retry rules?"
- "What's the auth provider? Token model? TTLs? Multi-tenancy if any?"

Where there's no current convention because everyone's doing whatever,
record that honestly — note it as an open question with the gap and a
proposed convention to discuss.

### Phase 7 — Business context

Goal: populate the relevant section of `README.md` (or a new
`business-context.md` if it gets long).

Ask:

- Customer / user — who uses this
- Stage / scale — rough order of magnitude (req/sec, users, revenue
  bracket)
- Constraints — compliance, regional, contractual commitments that
  bound architectural choices
- What success looks like at the company level over the next 6–12
  months

This phase informs roadmap conversations later.

### Phase 8 — Company information

Goal: populate `company-profile.md` (structured, official) and seed
`company-notes.md` (freeform, internal). Optional but high-value —
gives `/onboard` a strong starting picture for new collaborators.

For `company-profile.md`:

1. **Mission** — one paragraph. Why this company exists.
2. **Products** — one subsection per product line (status, audience,
   differentiator).
3. **Leadership** — founders, exec team, advisors. (Not on-call —
   that's `ownership.md`.)
4. **Mottos / pillars** — short load-bearing internal sayings.
5. **Strategic direction (current)** — current focus, bets we're
   making, bets we're declining.

For `company-notes.md`: skip the interview — this is the user's
free-form pad. Just confirm the file exists and ask if they want to
seed it with anything specific (recent thinking, observations).

For `events.md` Upcoming section: ask if there are imminent events
worth tracking (conference next month, demo, board meeting). Seed
the file with what they have; future events get added by hand
later.

## Output discipline

- **End every session with a commit-ready set of files.** Show the user
  the diff for each file you've drafted, ask for changes, then
  finalize.
- **At session end, write or update a `state/audit-progress.md`** with:
  - Which phases are complete
  - Which phases are partial (and what's missing)
  - Which phases haven't been started
  - Open questions surfaced this session
  - Where to resume

  Subsequent `/audit` runs read this file first.

- **Surface tension explicitly.** If the user's answers contradict each
  other, or contradict an existing file, name it: "you said X earlier;
  we wrote Y in `contracts/models.md`. Which is current?"

## Style rules

- **One question at a time** during interview phases. Don't batch.
- **No yes-or-no when an open answer is more useful.** "What are the
  core entities?" beats "do you have a User entity?".
- **Echo back before writing.** "So the API is Node 20 on Azure
  Container Apps with horizontal autoscaling on CPU at 70%, public
  ingress with 30s timeout — accurate?" Then write.
- **No marketing voice.** "Solid stack" / "great choices" — don't.
  This is documentation, not commentary.

## What you must NOT do

- **Don't autonomously scrape sub-repos to "discover" the truth** in
  Phase 1–4. Code can't tell you why something is the way it is. The
  user is the source. (Phase 5 endpoint discovery from code is fine if
  the user explicitly asks.)
- **Don't write to sub-repos** during the audit. The audit is purely
  populating orchestrator state.
- **Don't open migrations** during the audit. If the audit surfaces a
  needed change, file it as an open question; the user opens migrations
  via `/migration` later.
- **Don't push the user past their answer.** If they say "I don't know
  the cold-start behavior of the function app," accept it, file as an
  open question, move on.

## When NOT to use this skill

- **Adding one new fact** to existing populated state — just edit the
  file directly.
- **Filing a single decision** — use `/decision`.
- **Opening a migration** — use `/migration`.
- **Reading what's already populated** — use `/status` or `/onboard`.

## What "done" looks like for an /audit session

Either:

- A complete phase committed to its file(s) with the user's review, OR
- A partial phase explicitly marked partial in `state/audit-progress.md`
  with the next questions queued up.

Plus any open questions surfaced this session appended to
`open-questions.md`.

The audit as a whole is "done enough" when:

- Every section in `stack/`, `contracts/`, `conventions/`, and
  `platform-constraints.md` has either content or an explicit "N/A —
  reason" note.
- `state/manifest.md` lists every sub-repo.
- `open-questions.md` reflects current real questions, not the original
  audit gaps.

Then the orchestrator transitions from audit mode to operating mode,
where the dominant skills are `/migration`, `/status`, and `/decision`.
