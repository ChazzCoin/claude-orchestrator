# Changelog

## 0.10.0 — 2026-05-09

Skill-alignment sweep + new `/backlog` compiler. Every existing
skill is now reviewed for the v0.9.0 architecture (workspace-internal
sub-repos, scripted-skill discipline, output catalogue), updated for
stale references, given a pattern declaration, and cross-linked to
related new skills.

### New skill: `/backlog`

Cross-sub-project backlog compiler — third member of the compilation
trio.

| Skill | Scope |
|---|---|
| `/roadmap` | Future phases — strategic planning |
| `/backlog` | Queued items not yet started — tactical queue |
| `/tasks` | Active work (in-progress + next up) |

`/backlog` reads from claude-kit's `tasks/backlog/`, top-level
`BACKLOG.md`, or GitHub issues with the `backlog` label as
fallbacks. Sorts within each sub-repo by priority (high → medium →
low → unspecified), then by age. Flags items >60 days old as
`⚠ aging`. Surfaces cross-stack signals (priority gaps, coupling
opportunities, repos with no priority data). Output uses Pattern 4
(Sprint task board) for the artifact + Pattern 28 (Stats card grid)
for chat summary.

### Substantive updates to existing skills

- **`/brief`** — drops `state/manifest.md paths` references; switches
  to `repos/<name>/` working trees; adds inbox unread, events
  upcoming, company-notes recent-additions to the source list. Now
  explicitly the *delta* layer over `/status` (changed since last
  session vs current state).
- **`/onboard`** — canonical sources expanded to include
  `company-profile.md` and `events.md`. "How to operate" section
  reorganized into Daily / Compilers / Authoring / Setup groups
  showing all 19 skills. Adds prerequisite check for `repos/`
  population (suggests `bin/setup` first).
- **`/review`** — source pulls switched to `repos/<name>/` for git
  log queries (with `gh api` fallback for remote-only). Weekly mode
  pulls inbox + events + company-notes from the period. Quarterly
  mode reads `company-profile.md` strategic direction and runs
  `/roadmap`, `/tasks`, `/backlog` as part of strategic
  recalibration.
- **`/audit`** — new Phase 8 covers company information
  (`company-profile.md` interview, `company-notes.md` seed,
  `events.md` upcoming items). Optional but high-value for
  `/onboard` quality.

### Pattern declarations on every skill

Every `SKILL.md` now declares its output pattern(s) from
`output-catalogue.md`. The discipline (per
`kit/skill-architecture.md`) is: scripts declare in header
comments, skills declare inline near the top.

| Skill | Primary pattern(s) |
|---|---|
| `/audit` | 30 multi-step wizard, 26 empty state |
| `/backlog` | 4 sprint task board, 28 stats grid |
| `/brief` | 23 timeline, 28 stats grid |
| `/decision` | 14 decision tree, 1 hero card |
| `/feature` | 23 timeline, 1 hero card, 4 sprint board |
| `/inbox` | 23 timeline, 1 hero card |
| `/incident` | 25 alerts, 23 timeline, 1 hero card |
| `/migration` | 4 sprint board, 24 matrix, 1 hero card |
| `/onboard` | 30 wizard, 33 command reference |
| `/refresh` | 17 git branch overview |
| `/register` | 34 selection prompt, 1 hero card |
| `/review` | 28 stats grid, 23 timeline |
| `/risk` | 6 severity audit, 25 alerts |
| `/roadmap` | 3 roadmap timeline (multi-track), 28 stats grid |
| `/skills` | 33 command reference |
| `/status` | 28 stats grid + 17 git overview + 23 timeline (composed) |
| `/sync` | 23 timeline, 25 alerts, 1 hero card |
| `/sync-check` | 17 git overview, 6 severity audit |
| `/tasks` | 4 sprint board, 28 stats grid |

### Wiring updates

- `bin/init` — adds `state/backlog-compiled.md` to gitignore heredoc.
- `bootstrap/CLAUDE.md.template` — Skills section reorganized into
  Macro state / Sub-repo coordination / Compilation /
  Communication / Artifacts / Kit groupings. Adds the four new
  skills (`/refresh`, `/inbox`, `/roadmap`, `/tasks`, `/backlog`)
  with their roles.
- `README.md` — Skills shipped table updated with all 19 skills;
  notes the output-catalogue discipline.

### Deferred to v0.11.0

- **Convert `/sync` to scripted form** (`bin/sync`). The
  classify/copy/pin-bump flow is mechanical; skill should shrink to
  a thin wrapper that surfaces the script's diff report and
  proposes user-approved actions. Tracked in `/sync`'s SKILL.md.
- **Convert `/sync-check`, `/roadmap`, `/tasks`, `/backlog`,
  `/status` mechanical halves to scripts.** Each has data-gathering
  that should be deterministic (`bin/<name>-data` writing
  `state/<name>.json`); the skill renders.

## 0.9.0 — 2026-05-09

Two design shifts in this release: introduce the **scripted-skill
architecture** (skills can be locked-down shell scripts with thin
Claude wrappers, not all interpretive markdown contracts) and add the
**output catalogue** (34 visual patterns every script picks from,
never inventing custom shapes). Both shifts move toward
predictability over interpretation — Claude is great at judgment,
computers are better at deterministic behavior. The orchestrator
benefits from picking the right tool per task.

### Scripted-skill architecture

`bin/refresh` becomes the first scripted-skill prototype. The flow is
locked in shell: read manifest → fetch each cloned sub-repo →
compute ahead/behind/dirty → write `state/last-fetch.json` → print
summary. The `/refresh` skill drops from 160 lines to 47 — just "run
the script, surface output, don't reinterpret."

- `bin/refresh` (NEW, kit-synced, executable) — deterministic flow.
  Python stdlib for JSON output (no jq dependency). Tested
  end-to-end against rai-orchestrator's manifest.
- `kit/skills/refresh/SKILL.md` — rewritten as thin wrapper.
- `kit/skill-architecture.md` (NEW) — discipline doc covering:
  - Two shapes (scripted / interpretive / mixed) with decision rule.
  - Why scripting locks down behavior (lockdown, doc-by-code,
    user-runnable).
  - Configuration: scripts read fields from
    `.claude/local-config.json` (Python stdlib parsing).
  - Two layers: generic kit-synced (`bin/<name>`) vs templated
    instance-owned (`bootstrap/bin/<name>.sh.template`).
  - File-layout conventions, what NOT to script, migration path
    for converting existing skills.

### Output catalogue

`kit/output-catalogue.md` (NEW, 1501 lines) — visual design
catalogue with 34 terminal output patterns. Hero cards, live
dashboards, roadmap timelines, sprint boards, deployment reports,
severity audits, git branch overviews, activity timelines, alert
variants, empty states, stats grids, multi-step wizards, command
references, selection prompts, and more. Each pattern includes raw
template, color cues (semantic names), and design rationale.

Every orchestrator script picks a pattern from the catalogue based
on what its output is for. New requirements:

1. Declare the pattern in the script header comment.
2. Reference the pattern in the SKILL.md (if it has one).
3. Compose existing patterns; don't invent custom shapes.

Existing scripts retrofitted with pattern declarations:

- `bin/init` — Pattern 23 (Activity timeline) + Pattern 1
  hero-style next-steps block.
- `bin/setup` — Pattern 23 + summary block + one-time Pattern 34
  identity prompt on first run.
- `bin/refresh` — Pattern 17 (Git branch overview). Output rewritten:

```
  REFRESH  ·  2026-05-09 01:44 UTC

   ●  api       main                  +0 -3      behind origin
   ●  web       main                  +0 -0      current
   ●  ios       feat/x                +2 -0      ahead origin · dirty
   ○  devops    unfetched                        run bin/setup to clone
   ✗  api       (error)                          ERROR: <message>

   saved · state/last-fetch.json
```

  Glyphs: `●` cloned · `○` unfetched · `✗` error. Status text
  disambiguates: current / ahead / behind / diverged · dirty.

### Wiring

- `MANIFEST.json` — sync entries for `bin/refresh`,
  `kit/skill-architecture.md`, `kit/output-catalogue.md`.
- `bin/init` — copies `bin/refresh` into instances; copies the two
  new discipline docs to `.claude/`.
- `kit/orchestrator-rules.md` — session-start references list adds
  both new docs.

### Going forward

Every new script:

1. Picks a pattern from `output-catalogue.md`.
2. Declares the pattern in its header comment.
3. References the pattern in its SKILL.md (if any).
4. Composes existing patterns rather than inventing new shapes.

When a new pattern is genuinely needed, propose it as an addition to
`output-catalogue.md` via PR.

## 0.8.0 — 2026-05-08

Two big shifts in this release:

1. **Workspace-internal sub-repo model.** The orchestrator now owns
   its own checkouts under `repos/<name>/` rather than referencing
   external clones. Single mode, convention not configuration —
   sub-repos always live at `<orchestrator-root>/repos/<name>/`.
2. **Compilation primitives.** `/status` becomes a full cross-repo
   compiler. `/refresh`, `/inbox`, `/roadmap`, `/tasks` are new
   skills layered on top.

The two ship together because every compiler skill assumes
`repos/<name>/` is the canonical local view. The earlier
"register-where-they-live" model didn't work for multi-machine,
mobile, or fresh-collaborator flows. This release picks the simpler
answer: the orchestrator IS the workspace; sub-repos clone into it.

### Workspace-internal model

- `bootstrap/state/manifest.md.template` — drop "Local path" field
  from format spec, all entries, commented template; layout diagram
  rewritten for `repos/<name>/`; updated kit-enabled detection prose.
- `kit/state/sub-repos/_template.md` — drop `local_path` from
  frontmatter.
- `kit/sub-project-registration.md` — full rewrite. Single-mode,
  `/register` (per-repo, deliberate) + `bin/setup` (bulk, automatic)
  flows, mobile-fallback, edge cases, anti-patterns updated.
- `kit/sub-projects.md` — Registration headline rules rewritten;
  Git management → Fetching subsection introduces `/refresh` and
  session-start staleness.
- `kit/skills/register/SKILL.md` — full rewrite. Asks for
  `git_remote` instead of local path; verifies via `git ls-remote`;
  clones into `repos/<name>/` as the final step.
- `kit/skills/sync-check/SKILL.md` — uses `repos/<name>/`
  convention; remote-only fallback for machines without local
  clones; surfaces stale-fetch warnings.
- `kit/skills/migration/SKILL.md` — references
  `repos/<name>/.claude/active-migrations.md`; flags missing local
  clones for user resolution.

### New script: `bin/setup`

- Bulk-clones every registered sub-repo into `repos/<name>/`.
- Idempotent (skips already-cloned, flags collisions).
- Best-effort (reports per-repo failures without aborting).
- Prompts once for identity (`me`, `machine_id`) on first run if
  `.claude/local-config.json` is missing — otherwise silent.
- Mobile / remote-only environments skip it entirely; read-side
  skills fall back to `gh api` against the manifest's `git_remote`.

### Identity config

- `.claude/local-config.json` (NEW, gitignored) — per-machine
  config: `me`, `machine_id`, `inbox_last_read`. Set during
  `bin/setup` first run.

### New skills

- **`/refresh`** — fetches `origin` for every cloned sub-repo,
  records ahead/behind/dirty per repo, writes
  `state/last-fetch.json` so other skills know how stale the data
  is. Read-only against remotes; never auto-pulls. Working trees
  untouched.
- **`/inbox`** — per-person messaging. Send (writes to
  `state/inbox/<recipient>.md`, committed via git); read your own
  unread (filtered by per-machine `inbox_last_read`); list sent;
  list inboxes. Delivery is the recipient's next `git pull`.
- **`/roadmap`** — compiles cross-sub-project roadmaps into
  `state/roadmap-compiled.md`. Phase-based view showing what each
  team is planning side-by-side. Sources (priority): claude-kit's
  `tasks/ROADMAP.md`, top-level `ROADMAP.md`, README "Roadmap"
  section. Surfaces alignment / misalignment in chat summary.
- **`/tasks`** — compiles active-task state from each sub-repo
  into `state/tasks-compiled.md`. Active scope only (in-progress +
  next up; backlogs excluded). Flags stale tasks (≥7 days no
  commit on branch).

### Rewritten skill: `/status`

The load-bearing daily-driver compiler. Reads:

- Inbox unread count + headlines (for current user)
- Roadmap "Now" + "Recently shipped"
- Upcoming events (next 14 days)
- Active migrations with stale flags
- Open questions tagged Blocking
- Per-sub-repo state (branch / ahead / behind / dirty / sub-kit
  advertisement / open PRs) — from cached `state/last-fetch.json`
  and `state/sync-status.md`
- Drift (commits / branches not tied to any active migration)

No `gh` calls, no `git fetch` — pure compile from on-disk state.
Surfaces stale-fetch warnings when data is >24h.

### New bootstrap templates (instance-owned, skip-if-exists)

- `bootstrap/company-profile.md.template` — structured: mission,
  products, leadership, mottos, strategic direction.
- `bootstrap/company-notes.md.template` — freeform pad for
  half-formed thoughts, internal discussions, potential directions.
- `bootstrap/events.md.template` — Upcoming + Past sections for
  conferences, demos, meetings, launches.

### Other updates

- `kit/orchestrator-rules.md` — session-start checks list adds
  `state/last-fetch.json` with the 24h staleness rule.
- `bin/init`:
  - Gitignore heredoc adds `repos/`, `.claude/local-config.json`,
    `state/last-fetch.json`, `state/roadmap-compiled.md`,
    `state/tasks-compiled.md`.
  - Scaffold list adds `state/inbox`.
  - New `bootstrap_copy` entries for company-profile, company-notes,
    events.
  - Copies `bin/setup` into instances (with `chmod +x`).
- `MANIFEST.json` — adds sync entries for `bin/setup`,
  company-profile, company-notes, events; adds `state/inbox` to
  scaffold.
- `README.md` — tree shows `bin/setup`; new Instance layout and
  Onboarding a collaborator sections.

### Migration notes for existing instances

Existing instances that pre-date this release register sub-repos at
external absolute paths. To migrate:

1. `/sync` to bring kit changes downstream.
2. Strip `Local path:` lines from `state/manifest.md` and
   `local_path:` from `state/sub-repos/<name>.md` frontmatter.
3. Add `repos/`, `.claude/local-config.json`, and the new
   generated-state files to `.gitignore`.
4. Run `bin/setup` to clone registered sub-repos into
   `repos/<name>/`. First run prompts for identity.

Or simpler: re-bootstrap. The orchestrator's data (decisions,
migrations, ADRs, etc.) lives in committed bootstrap files; a fresh
`bin/init` preserves them and just updates kit machinery. Existing
external sub-repo checkouts can be kept as personal working copies —
the orchestrator no longer references them.

## 0.7.0 — 2026-05-08

Codify the sub-project registration workflow. Answers the
fundamental question: how does a sub-repo physically get
associated with the orchestrator?

**Decision:** keep repos where they live. The orchestrator records
absolute paths in the manifest; it doesn't move, clone, or symlink
existing repos. Recommended layout is orchestrator + sub-repos as
siblings under a parent dir (e.g. `~/ChazzCoin/`).

**New discipline doc (kit-managed):**

- `kit/sub-project-registration.md` — full workflow:
  - The four options considered (keep / clone-into-orch /
    symlink / remote-only) and why option 1 wins for v1
  - Recommended sibling layout with example tree
  - Three input cases for `/register`:
    1. Path exists, repo is local → validate + register
    2. GitHub URL but not yet cloned → suggest sibling
       destination, clone with explicit approval, then register
    3. Neither → ask the user; don't guess
  - What `/register` never does (move existing, delete checkout,
    auto-clone without approval, register without remote, register
    non-git directories)
  - Edge cases (multiple checkouts, worktrees, shared between two
    orchestrators, network drives, fresh repos)
  - Future-proofing — `git_remote` is the portable anchor that
    survives a future migration to remote-only (option 4)
  - Anti-patterns to avoid

**Updates:**

- `bootstrap/state/manifest.md.template` — adds a "Recommended
  layout" section with sibling-layout example tree; example entry
  shows sibling path.
- `kit/sub-projects.md` — new "Registration" section with headline
  rules referencing the new discipline doc.
- `kit/orchestrator-rules.md` — Sub-projects section gains a brief
  "Registration" subsection referencing the discipline doc.
- `bootstrap/CLAUDE.md.template` — operating-discipline file list
  gains the new doc.
- `MANIFEST.json` + `bin/init` — kit/sub-project-registration.md →
  .claude/sub-project-registration.md (file-replace, synced).

## 0.6.0 — 2026-05-08

Wire design standards as a tracked macro-state concern. Visual
identity, brand language, asset registry, collateral specs, per-
platform packages — all become orchestrator-tracked truth. PR
review gains a Design fit check.

**Discipline doc (kit-managed):**

- `kit/design.md` — what's tracked (tokens, brand, assets,
  collateral, per-platform), source-of-truth rules ("the
  orchestrator is the registry, not the design tool"), update
  cadence (token changes paired with ADR + maybe migration),
  validation pattern, what this discipline is NOT (not a Figma
  replacement, not a brand guidelines deliverable).

**Bootstrap templates (instance-owned, skip-if-exists):**

- `bootstrap/design/README.md.template` — directory overview
- `bootstrap/design/brand.md.template` — color palette (primary /
  neutral / semantic), typography (families + scale + weights),
  spacing, radii / shadows, motion, voice + tone, accessibility
  baseline. **Source of truth for tokens.**
- `bootstrap/design/assets.md.template` — asset registry: logos
  (with format / size variants), icons, illustrations,
  photography, local committed assets, sources outside this
  directory.
- `bootstrap/design/materials.md.template` — collateral: email
  signatures, business cards, letterhead, presentation templates,
  social media assets, swag, press kit.
- `bootstrap/design/mobile.md.template` — mobile graphics package:
  iOS + Android app icons (all sizes), splash / launch screens,
  in-app artwork (empty states, onboarding), mobile-specific
  tokens, App Store / Play Store assets.
- `bootstrap/design/web.md.template` — web component specs
  (buttons, inputs, cards, tables), page templates, layout grid,
  breakpoints, dark mode strategy, web accessibility.

**Validation:**

- `kit/pr-reviews/_template.md` — Design fit check added to the
  Macro contract fit section: tokens used vs hardcoded, palette
  match, typography match, asset registry conformance, per-
  platform conventions. Voice + tone check for user-facing copy.
- `kit/orchestrator-rules.md` — artifact map gains rows for design
  / brand changes (manual edit + ADR + maybe migration), asset
  version bumps, collateral / mobile / web spec changes; AUDIT
  emoji 🎨 added.
- `bootstrap/AUDIT.md.template` legend updated with 🎨.
- `bootstrap/CLAUDE.md.template` "Where things live" gains a
  Design / brand section pointing at the new files.

**Wiring:**

- `MANIFEST.json` — kit/design.md → .claude/design.md (file-
  replace); bootstrap/design/* → design/* (skip-if-exists).
- `bin/init` — copies design discipline doc + design bootstrap
  templates.

## 0.5.0 — 2026-05-08

Two hard rules. Sharpens what was implicit before.

**Rule 1 — Read-then-act on sub-projects.** When the user asks for
something involving a sub-project, the orchestrator reads the
sub-project's own docs first (its CLAUDE.md, runbooks, conventions,
ADRs) and then takes the matching path. Specifically:

- **Read queries** (commands that don't change state) — orchestrator
  runs them directly using sub-project docs as reference.
- **State-changing commands** — orchestrator runs them with explicit
  user approval per CLAUDE.md "Executing actions with care".
- **Non-running file edits** — orchestrator drafts in `chore/orch-*`
  PR.
- **Code changes** — file a task (kit-enabled), or user does it
  manually.

The orchestrator USES the sub-project's playbooks rather than
inventing new ones. If a runbook is missing or wrong, fix the
runbook first.

**Rule 2 — Code boundary (non-negotiable).** The orchestrator does
**NOT** modify running files without explicit user override.
"Running files" defined precisely: source code, build / packaging
configs, CI/CD pipelines, infrastructure-as-code, runtime configs,
schema migrations, generated code. The test: "does anything that
runs in production load this file?"

When asked for a code change:
1. Pause. Identify it as a code touch.
2. Advise against the orchestrator doing it — reasoning: skips
   sub-kit verification gate, lacks per-repo context, hand-edits
   to code are risky.
3. Suggest the right path: task spec (kit-enabled) or user manual.
4. Wait for explicit override.
5. If overridden — open `chore/orch-*` PR with `⚠ Code touch under
   user override` marker; AUDIT entry with 🚨; recommend running
   the verification command before merge.

**Wiring:**

- `kit/sub-projects.md` — three new sections: "Read-then-act
  protocol", "Running files vs non-running files" (with the test +
  edge cases), "Code boundary (non-negotiable)" (with the
  five-step override discipline). Path 2 of the Write protocol
  renamed from "Documentation / config / spec changes" to
  "Non-running file updates" to reflect the sharper boundary.
- `kit/orchestrator-rules.md` — Sub-projects section gains
  "Read-then-act", "Code boundary (non-negotiable)", and "Run —
  commands from sub-project docs" subsections; Path 2 in the
  four sanctioned paths sharpened; "What the orchestrator does
  NOT do" list led by the code-touch rule.
- `bootstrap/CLAUDE.md.template` — Hard rules section gets two new
  rules: Read-then-act on sub-projects, and Don't touch code.
- New AUDIT emoji: 🚨 for code touches under user override.

## 0.4.0 — 2026-05-08

Add the macro-grade PR review structure. The orchestrator's review
sits above sub-kit's per-repo `/review`: catches "does this PR fit
the contract" rather than "is this line correct." Skills are
unchanged in this release.

**Reference + template (kit-managed):**

- `kit/pr-reviews/README.md` — when to review (touches contract
  surface, active migration, conventions, vendors, auth, schemas;
  or orchestrator-authored), what to check (task alignment, code
  reads, naming/error/auth conventions, contracts, platform
  constraints, tech principles, cross-platform fit, gotchas, risk
  references), three outcomes (approved / conditional / request
  changes), what spawns artifacts (risks, ADRs, contract updates).
- `kit/pr-reviews/_template.md` — the structured review form.
  Frontmatter for PR URL, sub-repo, status, related artifacts;
  body has all the checklist sections + a draft PR comment for the
  user to approve before posting.

**Wiring:**

- `kit/orchestrator-rules.md`: artifact map gets a PR-review row;
  AUDIT emoji legend gets 🔍; new "PR review lifecycle (macro
  grain)" section parallel to migration / risk / incident.
- `bootstrap/AUDIT.md.template` emoji legend updated to include 🔍.
- `kit/sub-projects.md` "Git management" section gets a "Reviewing
  PRs (orchestrator-grade)" subsection — when to invoke, what's
  read, where output lands, never auto-approve / auto-merge.
- `kit/state/sub-repos/_template.md` adds a "Recent PR reviews on
  this repo" section. /sync-check populates it from `pr-reviews/`.
- `bootstrap/CLAUDE.md.template` quick-reference artifact map +
  "Where things live" updated with PR reviews.
- `MANIFEST.json` and `bin/init`: kit/pr-reviews/ → pr-reviews/
  (directory-mirror-files); pr-reviews/ scaffold dir.

**Discipline:** sub-kit's per-repo `/review` runs first (line-level
issues); orchestrator review runs second when macro context is
needed. The two compose; their outputs live separately. Action
items from a macro review spawn real artifacts (risks, ADRs) — a
review with no follow-up is theater.

## 0.3.0 — 2026-05-08

Wire the sub-project structure. Codifies how the orchestrator
relates to the codebases it tracks — kit-enabled vs non-kit, what's
read, what's written, git discipline, sub-kit advertisement
protocol. Skills are unchanged in this release; the structure is
the foundation skills will use.

**Reference docs (kit-managed, synced via /sync):**

- `kit/sub-projects.md` — the master sub-project document. Two
  flavors (kit-enabled, non-kit), read protocol, four sanctioned
  write paths, sub-kit advertisement protocol, git management
  discipline. Codifies the load-bearing rule: **tasks are for
  coding work; doc/config/spec changes the orchestrator does
  directly via PR.**
- `kit/claude-kit-reference.md` — pinned reference for claude-kit's
  shape (top-level layout, key files, state machine, branch / PR
  conventions). Re-synced manually when claude-kit changes.

**State shape (kit-managed templates + scaffold):**

- `kit/state/sub-repos/_template.md` — per-sub-repo state file
  template. Each registered sub-repo gets
  `state/sub-repos/<name>.md` populated from this. Holds: current
  state (auto), capabilities (auto), affecting-this-repo references
  to all orchestrator artifacts (auto), advertisement (auto), Notes
  (manual, preserved across sync).
- `kit/state/README.md` updated to document the manifest-vs-depth
  split and the auto vs manual fields.
- `kit/templates/sub-repo-advertisement/active.md.template` + README
  — the voluntary sub-kit advertisement protocol shape.

**Manifest format (bootstrap, instance-owned):**

- `bootstrap/state/manifest.md.template` extended with
  `Kit-enabled`, `Last synced HEAD`, `Last synced at`, and
  `State file` (link to per-sub-repo file) fields. The manifest
  becomes a true index pointing into per-sub-repo depth files.

**Operating discipline updates (kit-managed):**

- `kit/orchestrator-rules.md` "Sub-repo write discipline" replaced
  with "Sub-projects — read / write protocol" — references
  sub-projects.md, summarizes the four write paths, summarizes git
  discipline. Old "exactly one place" framing removed (the
  orchestrator can write doc PRs and task specs too, just through
  different paths).
- `bootstrap/CLAUDE.md.template` updated to point at all three
  kit-managed reference docs (orchestrator-rules.md,
  sub-projects.md, claude-kit-reference.md) and summarize the four
  write paths.

**Wiring:**

- `MANIFEST.json` extended with new kit entries (sub-projects.md,
  claude-kit-reference.md, state/sub-repos/_template.md) and
  `state/sub-repos` scaffold dir.
- `bin/init` mirrors the new kit files and scaffolds
  `state/sub-repos/`.

## 0.2.0 — 2026-05-08

Scaffold the CTO operating model. Adds the structural analog of
claude-kit's task-rules + lifecycle for the macro level. Skills for
the new artifact types are stubs — behavior contracts get refined on
first real use.

**Operating discipline (new)**
- `kit/orchestrator-rules.md` — generic CTO operating model: read
  order, artifact map, audit log discipline, migration / risk /
  incident / review lifecycles, sub-repo write rules, decision
  authority, closing-artifact rule. Synced via `/sync` (the
  orchestrator's analog of claude-kit's `task-rules.md`).
- Bootstrap CLAUDE.md.template slimmed — defers operating discipline
  to `orchestrator-rules.md`; keeps role + hard rules + quick
  reference.

**Artifact types (new)**
- `kit/risks/` — risk register pattern with severity × likelihood
  matrix. Instances get `risks/{open,mitigated}/`.
- `kit/incidents/` — cross-stack postmortem pattern + 48h rule +
  action-items-must-become-artifacts discipline.
- `kit/reviews/` — recurring rhythm: weekly / monthly / quarterly
  templates with cadence docs. Instances get
  `reviews/{weekly,monthly,quarterly}/`.

**Skills (stubs)**
- `/risk` — file / update / list / mitigate / accept
- `/incident` — open / update / postmortem / close
- `/review` — weekly / monthly / quarterly with synthesized state pulls
- `/brief` — synthesized CTO orientation across sub-repos +
  orchestrator artifacts

**Bootstrap (instance-owned, new)**
- `tech-vision.md.template` — 12–18mo direction (anchor)
- `tech-principles.md.template` — decision criteria (anchor)
- `vendors.md.template`, `ownership.md.template`, `slos.md.template`,
  `cost-tracking.md.template`, `security-posture.md.template`,
  `compliance.md.template` — operational registries
- `AUDIT.md.template` — chronological macro log

**Wiring**
- `MANIFEST.json` extended with new kit + bootstrap entries and
  `risks/open`, `risks/mitigated`, `reviews/weekly`,
  `reviews/monthly`, `reviews/quarterly` scaffold dirs.
- `bin/init` mirrors the new kit dirs, copies the new bootstrap
  templates, scaffolds the new dirs, and gitignores
  `state/brief-cache.md` for the future `/brief` skill.

## 0.1.0 — 2026-05-07

Initial restructure to claude-kit-style template.

- `kit/` (synced via `/sync`) holds skills + generic ADR / feature /
  migration templates and format docs.
- `bootstrap/` holds one-time `.template` files copied at init —
  CLAUDE.md, README.md, roadmap.md, open-questions.md,
  platform-constraints.md, stack/, contracts/, conventions/,
  state/manifest.md, foundation.json.
- `bin/init` bootstraps a `<company>-orchestrator` instance.
- `MANIFEST.json` declares all file mappings and policies.
- New skills: `/sync` (kit-pull), `/skills` (lister).
- Top-level `CLAUDE.md` removed — match claude-kit. Behavior contract
  lives only at `bootstrap/CLAUDE.md.template` for instances.
- Generated state (`state/sync-status.md`, `state/audit-progress.md`)
  is gitignored in instances.

## 0.1.1 — 2026-05-07

- `kit/templates/sub-repo-notices/migrations.md.template` codifies
  cross-repo write path. Instance gets `.claude/templates/`.
- `/migration` references the template for wholesale-regenerate
  semantics on open and close.
