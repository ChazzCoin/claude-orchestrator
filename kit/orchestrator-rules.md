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
- [`state/last-fetch.json`](state/last-fetch.json) — timestamp of the last `/refresh` against sub-repo remotes. If missing or older than 24h, surface that the picture may be stale and suggest `/refresh` before any compiler skill (`/status`, `/sync-check`, `/roadmap`, `/tasks`). Don't refuse to run those skills; just warn.
- [`.claude/skill-architecture.md`](.claude/skill-architecture.md) — discipline for scripted vs interpretive vs mixed skills. Read when designing a new skill or converting one. Scripted skills are thin wrappers around `bin/<name>` shell scripts; the script is the spec.
- [`.claude/output-catalogue.md`](.claude/output-catalogue.md) — visual design catalogue (34 patterns) for terminal output. Every script's output picks a pattern; never invent a custom shape. Read when building a new script or designing what a skill displays.

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
| PR review at the macro grain (does it fit the contract?) | Review file in `pr-reviews/` | `/pr-review` |
| Sub-repo registered or deregistered | `state/manifest.md` entry | `/register` |
| New question / observation, deferred decision | Entry in `open-questions.md` | manual edit |
| Strategic direction shift | `roadmap.md` update | manual edit |
| Long-arc tech vision shift | `tech-vision.md` update | manual edit + ADR |
| Tech principle change | `tech-principles.md` update | manual edit + ADR |
| Design / brand change (token, palette, typography, voice) | `design/brand.md` update | manual edit + ADR (+ migration if cross-repo) |
| Design asset version bump | `design/assets.md` update | manual edit |
| Collateral / mobile / web spec change | `design/{materials,mobile,web}.md` update | manual edit |
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
- 🔍 PR review filed (macro grain)
- 🚀 Cross-repo deploy / coordinated release
- 🎯 Quarterly review completed
- 🔧 Macro state refreshed (e.g. `/audit` Phase N landed)
- 🤝 Sub-repo registered / deregistered
- 🚨 Code touch under user override (orchestrator hand-edited a running file)
- 🎨 Design / brand / asset change (token, asset version, collateral spec)

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

## PR review lifecycle (macro grain)

Full discipline lives in [`pr-reviews/README.md`](pr-reviews/README.md). Headline:

- **Not every PR.** Only when: touches an active migration, touches
  a contract surface, touches conventions / vendors / auth /
  schemas, or was authored by the orchestrator itself.
- **Three outcomes:** ✅ approved · ⚠ conditional · ❌ request changes.
- **Two artifacts may follow:** a posted PR comment (via `gh pr
  comment`, with user approval) and any spawned artifacts (risks,
  ADRs, contract updates) the review surfaced.
- **Never auto-merge or auto-approve.** Reviews surface; user decides.
- **Different from sub-kit `/review`** — that's per-repo line-level;
  this is macro contract fit. The two compose; they don't replace.

AUDIT entry with 🔍 when a review is filed.

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

## Sub-projects — read / write protocol

Full discipline lives in [`sub-projects.md`](sub-projects.md).
Headline rules:

### Two flavors

- **Kit-enabled** — has `<sub>/.claude/foundation.json`. Orchestrator
  reads richly (CLAUDE.md, tasks/, AUDIT.md, advertisement). Format
  reference: [`claude-kit-reference.md`](claude-kit-reference.md).
- **Non-kit** — orchestrator gets git-only treatment. Offer to
  install claude-kit once at registration; record decline in
  manifest notes.

The orchestrator is **agnostic** — works with both. Kit-enabled is
richer, non-kit is sufficient for migration coordination.

### Read

| Source | When |
|---|---|
| `git log`, `gh pr list`, `git status` | every `/sync-check`, before any write |
| `<sub>/.claude/foundation.json` | register, sync — kit detection |
| `<sub>/.claude/active.md` | sync — voluntary advertisement |
| `<sub>/CLAUDE.md` | register — confirm orchestrator back-pointer |
| `<sub>/tasks/`, `<sub>/docs/` | on demand (kit-enabled only) |

The orchestrator NEVER edits files inside a sub-repo without going
through git (branch + PR + user-approved merge).

### Read-then-act

When the user asks for something involving a sub-project:

1. Read the sub-project's docs first (its CLAUDE.md, runbooks,
   conventions).
2. Classify: read query / state-changing command / non-running
   file update / **code change**.
3. Route per the matching path below.

### Write — four sanctioned paths *(running files are off-limits without override)*

1. **Migration notices** — `<sub>/.claude/active-<concern>.md`.
   Owned by `/migration` (and future per-concern skills). Templates
   in [`templates/sub-repo-notices/`](templates/sub-repo-notices/).
2. **Non-running file updates** — orchestrator drafts the change
   directly, opens a PR (`chore/orch-<YYYY-MM-DD-slug>`), user
   approves the merge. Use for: doc updates (`README`, runbooks,
   ADRs in `<sub>/docs/`), phase additions, ROADMAP edits, AUDIT
   entries, CLAUDE.md corrections, conventions docs. **Never for
   running files** (source code, build configs, CI, infra-as-code,
   runtime configs) — see Code boundary below.
3. **Coding-task specs** — orchestrator drafts a task spec into
   `<sub>/tasks/backlog/TASK-NNN-slug.md` (kit-enabled only),
   opens a PR with the spec. The sub-kit does the actual coding.
4. **claude-kit install** — orchestrator runs claude-kit's
   `bin/init` against the sub-project (only with user approval —
   typically once at registration).

**The split is load-bearing:** tasks are for **coding work**.
Non-running file edits the orchestrator does itself via PR.
Running files are off-limits.

### Code boundary (non-negotiable)

The orchestrator does **NOT** modify running files (source code,
build configs, CI/CD, infra-as-code, runtime configs, anything
loaded by a service at runtime) without **explicit user override**.

When the user asks for a code change:

1. **Pause.** Identify the file as running.
2. **Advise against the orchestrator doing it.** Reasoning: the
   orchestrator skips the sub-kit's verification gate / test
   pairing / per-repo context. Hand-edits to code are risky.
3. **Suggest the right path** — task spec (kit-enabled) or the
   user does it manually.
4. **Wait for explicit override.** Anything ambiguous → re-ask.
5. **If overridden** — open the PR with `⚠ Code touch under user
   override` in the body; AUDIT entry flags the override
   explicitly; recommend the user run the verification command
   before merging.

Full discipline at [`sub-projects.md`](sub-projects.md) "Code
boundary".

### Run — commands from sub-project docs

The orchestrator can run commands sourced from a sub-project's
documentation (its runbooks, CLAUDE.md, ADRs):

- **Read queries** (no state change) — run directly.
- **State-changing commands** — run with **explicit user approval**
  per CLAUDE.md "Executing actions with care". This applies to
  deploys, scales, restarts, secret rotations, etc.

The orchestrator USES the sub-project's playbooks rather than
inventing new ones. If a runbook is missing or wrong, fix the
runbook (path 2: non-running file update) before running anything
that depends on it.

### Git discipline (every write)

- **Pull before write.** `git fetch` + verify clean working tree;
  abort if dirty.
- **Branch from up-to-date main.** Never write to main directly.
- **PR title prefix `orch:`** so sub-repo's PR list distinguishes
  orchestrator-authored work from sub-kit work.
- **Body links to motivating orchestrator artifact** (ADR ID,
  migration ID, etc.).
- **Never auto-merge.** User approves every merge.
- **Don't bypass hooks** unless user explicitly asks.

Full rules: [`sub-projects.md`](sub-projects.md) "Git management".

### Tracking

Per-sub-repo state lives at `state/sub-repos/<name>.md`. The
manifest at `state/manifest.md` is the index; the per-sub-repo
file is the depth. See [`state/README.md`](state/README.md).

### Registration

How sub-repos physically land on disk and get associated with the
orchestrator is documented at
[`sub-project-registration.md`](sub-project-registration.md).
Headline: the orchestrator owns its checkouts at
`<orchestrator-root>/repos/<name>/`. `/register` is the per-repo,
deliberate first-time flow; `bin/setup` is the bulk, automatic
collaborator-onboarding flow. Path is convention, not configuration —
identical on every machine. Existing checkouts elsewhere on a user's
machine are not referenced by the orchestrator.

### What the orchestrator does NOT do

- **Modify running files (code) without explicit user override.**
  See "Code boundary" above. This is the hardest rule.
- Dispatch tasks to sub-kits. The user dispatches.
- Write to a sub-repo without a PR (except `active-*.md` notices,
  which are the structured one-way signal).
- Auto-merge any PR.
- Run state-changing commands without user approval.
- Invent commands when the sub-project's docs already have them.

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
