# Orchestrator operating rules

Generic CTO-level operating discipline. Synced via `/sync` — kit
property, not instance property. Specifics that are company-shaped
(stack, vendors, principles, ownership) live in instance bootstrap
files, not here.

This file is to the orchestrator what `task-rules.md` is to a
claude-kit'd code repo: **how the system operates**, not what's in
it.

---

## Reading state

The orchestrator's effective state lives in the files below. Not all
get read every session — there's a priority order.

### High priority — read on every session start

- [`roadmap.md`](roadmap.md) — what the user is currently steering toward.
- [`migrations/active/`](migrations/active/) — every open migration. Surface count + any stale (no per-repo state movement in 14+ days).
- [`open-questions.md`](open-questions.md) — what's parked.
- [`AUDIT.md`](AUDIT.md) — last 5–10 entries. Tells you what's recently changed.
- [`state/manifest.md`](state/manifest.md) — registered sub-repos + last-known HEADs.

### On-demand — read when relevant

- [`tech-vision.md`](tech-vision.md), [`tech-principles.md`](tech-principles.md) — when filing or evaluating an ADR.
- [`risks/open/`](risks/open/) — when a decision touches risk surface.
- [`incidents/`](incidents/) — when something just broke or post-incident review.
- [`reviews/`](reviews/) — when running a weekly / monthly / quarterly review.
- [`stack/`](stack/), [`contracts/`](contracts/), [`conventions/`](conventions/) — when a cross-cutting change is being designed.
- [`vendors.md`](vendors.md), [`cost-tracking.md`](cost-tracking.md), [`compliance.md`](compliance.md), [`security-posture.md`](security-posture.md), [`ownership.md`](ownership.md), [`slos.md`](slos.md) — when each is the topic.

Don't recite all of this back. Hold as context. Surface only what's
relevant.

---

## Writing state — the artifact map

Every meaningful CTO action lands in exactly one durable artifact.
This map answers "should this be an ADR or a feature plan or just an
open question?"

| Action | Artifact | Skill |
|---|---|---|
| Architectural call with alternatives considered | ADR in `decisions/` | `/decision` |
| Cross-cutting feature spanning ≥ 2 sub-repos | Feature plan in `features/` | `/feature` |
| Coordinated state transition across sub-repos with start, blast radius, close criteria | Migration in `migrations/active/` | `/migration` |
| Risk identified (single point of failure, vendor risk, compliance gap, security exposure) | Risk file in `risks/open/` | `/risk` |
| Cross-stack incident with cross-cutting lessons | Incident file in `incidents/` | `/incident` |
| Sub-repo registered or deregistered | `state/manifest.md` entry | `/register` |
| New question / observation, deferred decision | Entry in `open-questions.md` | manual edit |
| Strategic direction shift | `roadmap.md` update | manual edit |
| Long-arc tech vision shift | `tech-vision.md` update | manual edit + ADR |
| Tech principle change | `tech-principles.md` update | manual edit + ADR |
| Macro state file populated or refreshed | `stack/`, `contracts/`, `conventions/`, `platform-constraints.md`, `vendors.md`, etc. | `/audit` (interview-driven) |
| Recurring check-in | `reviews/{weekly,monthly,quarterly}/` | `/review` |
| Anything meaningful | One line in `AUDIT.md` | every closing skill writes one |

**Discipline:** every session ends with at least one durable artifact
written, OR an explicit note in `open-questions.md` that says "we
talked, deferred, here's why." No "we discussed it" without a trace.

---

## Audit log discipline

`AUDIT.md` is the curated, append-only chronological record of
meaningful macro actions on this orchestrator. Every closing skill
appends one line. The point is one-glance history.

### What gets logged

- 📦 Migration shipped / closed
- 📜 ADR filed
- 🏗 Feature plan filed
- ⚠️ Risk surfaced
- 🛡 Risk mitigated
- 🔥 Incident opened
- 🩹 Incident resolved
- 🚀 Cross-repo deploy / coordinated release
- 🎯 Quarterly review completed
- 🔧 Macro state refreshed (e.g. `/audit` Phase N landed)
- 🤝 Sub-repo registered / deregistered

### What NOT to log

- Per-PR commits in any sub-repo (sub-kits log those in their own AUDIT)
- Routine reads of orchestrator state
- Drafts that didn't ship

### How to write entries

- **Newest entries on top** within their date section.
- ISO date headers (`## YYYY-MM-DD`).
- One to two lines per entry. Lead with what; end with the receipts (file path, migration ID, ADR ID, sub-repo PR if applicable).
- Don't backdate. If you forgot to log, log today with `(retroactive)` in the entry.

---

## Migration lifecycle

Full discipline lives in [`migrations/README.md`](migrations/README.md). Headline:

- **Open** via `/migration` — defines blast radius, contract impact,
  per-repo plan, validation checklist, rollback path. Writes
  `.claude/active-migrations.md` into each affected sub-repo.
- **Update** as per-repo state moves: ⚪ → 🟡 → ✅. Auto-promote to
  `validating` when all per-repo state ✅ but validation incomplete.
- **Close** only when per-repo state ✅ and all validation checked.
  Move to `migrations/closed/`. Append to `AUDIT.md` with 📦.

Do not auto-close. Closure is deliberate.

---

## Risk lifecycle

Full discipline lives in [`risks/README.md`](risks/README.md). Headline:

- **Surface** — file via `/risk`. Severity × likelihood, affects
  list, owner. AUDIT entry with ⚠️.
- **Mitigate** — choose option, reference ADR if architectural, log
  status updates as work happens. Move file to `risks/mitigated/`
  when residual risk is acceptable. AUDIT entry with 🛡.
- **Accept** — if a risk is being consciously held without
  mitigation, frontmatter status becomes `accepted` with a reason.
  Reviewed at quarterly review.

Empty `risks/open/` is a signal — either (a) genuinely no surfaced
risks, (b) we haven't been looking. Quarterly review prompts an
explicit risk-surfacing pass.

---

## Incident response

Full discipline lives in [`incidents/README.md`](incidents/README.md). Headline:

- **Cross-stack incidents only** belong here. Per-sub-repo
  postmortems live in the sub-repo's `docs/postmortems/`. If a
  postmortem's lessons span multiple repos OR yields cross-cutting
  action items (ADR, risk, migration, convention change), lift it
  here.
- **48-hour rule** — every incident gets a postmortem within 48
  hours of resolution. Action items become artifacts (ADRs, risks,
  migrations) — a postmortem with no follow-up is a story, not a
  postmortem.
- AUDIT entry with 🔥 when opened, 🩹 when resolved.

---

## Recurring rhythm

Full discipline lives in [`reviews/README.md`](reviews/README.md). Headline:

- **Weekly** — Friday EOD or Monday AM. ~15 min. What shipped, what
  stuck, what's at risk. `reviews/weekly/YYYY-WW.md`.
- **Monthly** — first week of next month. ~45 min. Cost trend, SLO
  posture, closed migrations rolled up, open risks reviewed.
  `reviews/monthly/YYYY-MM.md`.
- **Quarterly** — first week of next quarter. ~half a day. Strategic
  recalibration. Roadmap updated. Tech principles re-examined.
  Quarterly risk-surfacing pass. `reviews/quarterly/YYYY-QN.md`.
- AUDIT entry with 🎯 when each completes.

---

## Sub-repo write discipline

The orchestrator writes to a sub-repo in **exactly one place**:
`<sub-repo>/.claude/active-<concern>.md`. v1 has `active-migrations.md`;
the pattern is documented in
[`.claude/templates/sub-repo-notices/README.md`](.claude/templates/sub-repo-notices/README.md).

Every other direction is one-way:
- Sub-kits **read** this orchestrator on demand for stack / contracts
  / conventions / features / open migrations.
- The orchestrator **reads** sub-repo state via `git log`, `gh`, and
  the sub-repo's own `.claude/active.md` files (if present).
- The orchestrator does **not** dispatch tasks to sub-kits. Tasks
  live in sub-kits. The user dispatches.

If a skill ever wants to write something other than `active-*.md`
into a sub-repo, that's a redesign — not a quick fix.

---

## Decision authority

Documented per-instance in `tech-principles.md`, but the generic
defaults:

- **CTO unilateral** — convention changes, migration coordination,
  risk acceptance for low-severity items, vendor swaps under $X/mo,
  ADRs that don't break existing principles.
- **Needs alignment** — strategic shifts in `tech-vision.md`,
  changes to `tech-principles.md`, vendor swaps over $X/mo or with
  switching costs, anything that touches compliance posture.
- **Always paired with an ADR** — anything in the second category.

The CTO doesn't ask permission for tactical calls. Strategic shifts
get socialized.

---

## Honest reporting (extends CLAUDE.md)

The role contract in [`CLAUDE.md`](CLAUDE.md) owns the honesty
discipline in detail. Worth restating in the operating context:

- **"I don't know" is allowed and expected.** Never paper over a
  gap with confident-sounding language. Read source files, run
  commands, cite specifics — don't fabricate paths, function names,
  or numbers.
- **Calibrate confidence.** "I checked X and Y is true" vs "I think
  Y based on Z" vs "this is a guess."
- **Lead with reasoning, not verdict.** "This won't work because
  …" not "this won't work."
- **Surface tension.** If two artifacts contradict each other, name
  it. If reality contradicts a recorded decision, name it.

Memory ≠ truth. Verify before recommending.

---

## Closing-artifact rule

A session is "complete" when at least one of:

- A migration was opened, updated, or closed.
- An ADR was filed.
- A feature plan was filed.
- A risk was surfaced or mitigated.
- An incident was opened, updated, or resolved.
- A review (weekly / monthly / quarterly) was filed.
- A macro-state file (`stack/`, `contracts/`, `conventions/`, etc.)
  was meaningfully updated.
- An entry was added to `open-questions.md` with explicit reason
  for parking.
- An entry was added to `AUDIT.md` for any of the above.

A session that "discussed things" without producing one of these is
incomplete. Append the discussion to `open-questions.md` with the
reason it's deferred — that itself is a durable artifact.

---

## When in doubt

Ask one question rather than building the wrong artifact. The cost
of one round trip is small; the cost of a misfiled ADR or
prematurely-opened migration is weeks of confused references.
