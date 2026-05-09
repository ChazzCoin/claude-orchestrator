# claude-kit reference

Reference documentation for claude-kit's file shape, lifecycle, and
conventions. The orchestrator reads kit-enabled sub-projects with
confidence because claude-kit's structure is known — this document
pins that knowledge so any agent acting as the orchestrator can
re-read it cold.

> **Source of truth** is the live claude-kit repo at
> <https://github.com/ChazzCoin/claude-kit>. This file is a
> reference snapshot — re-sync it manually when claude-kit's shape
> changes meaningfully. The "Updating this reference" section at
> the bottom describes the discipline.

**Last synced from claude-kit:** *(populate when first synced)*
**Pinned to commit:** *(populate)*

---

## How the orchestrator detects a kit-enabled sub-project

A sub-project is kit-enabled when it has
`<sub>/.claude/foundation.json` and that file's `kit.repo` field
contains `claude-kit` (or matches the kit URL pattern). See
[`sub-projects.md`](sub-projects.md) "Two flavors" for the full
discipline.

---

## Top-level layout in a kit-enabled sub-project

After `claude-kit/bin/init` runs against a target, the target has
this shape:

```
<sub-repo>/
├── CLAUDE.md                       # project-specific behavior contract
├── tasks/
│   ├── backlog/                    # task specs not yet active
│   ├── active/                     # currently being worked
│   ├── done/                       # shipped (PR merged)
│   ├── PHASES.md                   # phases + scope paragraphs + task lists
│   ├── ROADMAP.md                  # forward-looking task list per phase
│   └── AUDIT.md                    # chronological log of meaningful actions
├── docs/
│   ├── decisions/                  # per-repo ADRs
│   └── postmortems/                # per-repo incident postmortems
└── .claude/
    ├── foundation.json             # kit pin (kit repo, branch, sha, last_synced)
    ├── task-rules.md               # universal execution rules (synced from kit)
    ├── task-template.md            # spec template for new tasks (synced from kit)
    ├── ios-task-rules.md           # platform extension (if iOS work)
    ├── web-task-rules.md           # platform extension (if web work)
    ├── ios-conventions.md          # platform conventions reference (if iOS)
    ├── active-migrations.md        # written by orchestrator's /migration
    ├── active-features.md          # written by orchestrator's /feature
    ├── active.md                   # voluntary advertisement to orchestrator
    ├── shared/                     # durable two-way per-repo context
    │   ├── architecture.md         # living architecture doc
    │   ├── inbox.md                # repo-bound message thread
    │   ├── notes.md                # freeform shared memory
    │   └── references.md           # curated URLs
    └── skills/                     # slash commands (synced from kit)
        ├── audit/
        ├── backlog/
        ├── build/
        ├── decision/
        ├── ios-release/            # platform-specific
        ├── onboard/
        ├── plan/
        ├── postmortem/
        ├── release/
        ├── review/
        ├── roadmap/
        ├── run/
        ├── schema-check/
        ├── skills/
        ├── spec-phase/
        ├── status/
        ├── stuck/
        ├── sync/
        ├── task/
        └── update-docs/
```

Files marked "synced from kit" come from claude-kit's `kit/`
directory and propagate via the sub-kit's `/sync` command. Bootstrap
files (CLAUDE.md, PHASES.md, ROADMAP.md, AUDIT.md) are written once
at init and owned by the project after.

---

## Key files the orchestrator cares about

### `<sub>/.claude/foundation.json`

The kit's pin. Detects kit-enabled status. Format:

```json
{
  "kit": {
    "repo": "https://github.com/ChazzCoin/claude-kit",
    "branch": "main"
  },
  "pinned_sha": "<sha>",
  "last_synced": "YYYY-MM-DD",
  "overrides": []
}
```

If this file exists with `kit.repo` matching `claude-kit`, the
sub-repo is kit-enabled.

### `<sub>/CLAUDE.md`

Per claude-kit's `bootstrap/CLAUDE.md.template` (with the
orchestrator-integration changes), this file should have a
**`## Macro architecture (orchestrator)`** section with:

```
**Orchestrator path:** /absolute/path/to/<company>-orchestrator
```

The orchestrator confirms this back-pointer at registration. If
absent or stale, the sub-kit may not be reading orchestrator
notices on session start — flag it.

CLAUDE.md also declares the project's **platform**
(`ios | web | python | android | backend | universal`) — used by
the sub-kit's skills (especially `/release`) for routing.

### `<sub>/tasks/`

The task system. Three states:

```
tasks/backlog/  →  tasks/active/  →  tasks/done/
```

Sub-kit transitions tasks via `git mv`. Orchestrator reads to know
which phase the sub-repo is in and what's currently active.

#### Task spec format

The orchestrator drafts a task spec into `tasks/backlog/` when the
user decides on a **coding** change in the sub-repo (per
[`sub-projects.md`](sub-projects.md) "Coding changes — task spec").

Filename: `tasks/backlog/TASK-NNN-short-slug.md`. Body shape per
claude-kit's `task-template.md`:

- Title (descriptive, matches filename slug)
- One-sentence user story / what
- One-line "why"
- Files expected to change
- Acceptance criteria (checklist)
- Test pairing (per project's convention from CLAUDE.md)

The orchestrator's task spec drafts should reference back to the
motivating orchestrator artifact (ADR, feature, migration, risk,
incident) in the spec body.

### `<sub>/tasks/PHASES.md`

Phase definitions. Each phase has a name, a 2–4 sentence scope
paragraph, and an ordered list of tasks. The orchestrator may
**add a phase here directly** via PR (it's a doc edit, not code
work) when a feature plan or migration introduces a new
significant scope.

### `<sub>/tasks/ROADMAP.md`

Forward-looking task list per phase. Format per claude-kit's
`task-rules.md` "Phase structure":

```
## Phase N: <noun phrase>

<scope paragraph>

- TASK-NNN — <title>
- TASK-NNN — <title>
```

When the orchestrator drafts a task spec, it should also add the
task line to ROADMAP.md under the appropriate phase (in the same
PR).

### `<sub>/tasks/AUDIT.md`

The sub-kit's chronological audit log. Format per claude-kit's
`task-rules.md` "Audit log":

- Newest entries on top within date sections
- ISO date headers (`## YYYY-MM-DD`)
- Emoji set: 🚀 release, 📦 task ship, 📜 rule change, 🏗
  scaffolding, 🔥 hotfix, ⚠️ incident/tradeoff

The orchestrator reads this to understand what's recently happened
in the sub-repo. The orchestrator's own `AUDIT.md` (at the
orchestrator level) is a parallel log for macro-level events; the
two don't merge.

### `<sub>/.claude/active-migrations.md`

Written by the orchestrator's `/migration` skill. Format spec at
[`kit/templates/sub-repo-notices/migrations.md.template`](templates/sub-repo-notices/migrations.md.template).
Sub-kit reads on session start per claude-kit's `task-rules.md`
"Active orchestrator notices" rule.

### `<sub>/.claude/active-features.md`

Written by the orchestrator's `/feature` skill. Format spec at
[`kit/templates/sub-repo-notices/features.md.template`](templates/sub-repo-notices/features.md.template).
Lists open cross-cutting features (`status: planning` or
`in-progress`) whose `affects:` includes this repo. Same
auto-managed discipline as `active-migrations.md` — regenerated
wholesale, deleted on empty. Sub-kit reads on session start.

### `<sub>/.claude/shared/`

Durable two-way per-repo context channel. Different discipline
from `active-<concern>.md`: hand-edited (or appended), bidirectional
(both orchestrator and sub-kit write), never auto-regenerated.
Templates at [`kit/templates/sub-repo-shared/`](templates/sub-repo-shared/);
README at [`kit/templates/sub-repo-shared/README.md`](templates/sub-repo-shared/README.md).
Sub-kit reads `shared/inbox.md` and `shared/notes.md` on session
start per the dev pre-flight checklist
([`kit/governance/dev-preflight.md`](governance/dev-preflight.md)
step 2).

### Orchestrator-side proposal staging *(not in the sub-repo)*

The orchestrator may stage proposed phases and tasks for this repo
at `proposals/<this-repo>/` in the orchestrator before committing
them to `<sub>/tasks/`. The sub-kit doesn't see this directly —
proposals appear in the sub-repo only at promotion time, as a
standard `chore/orch-task-NNN-<slug>` or `chore/orch-phase-<id>`
PR. The promoted task spec body conforms to the sub-kit's
`task-template.md` shape; reference back to the proposal lives in
the PR body.

Reference: [`../proposals/README.md`](../proposals/README.md).

### `<sub>/.claude/active.md`

Voluntary advertisement from the sub-kit to the orchestrator. Read
by `/sync-check`. Format spec at
[`kit/templates/sub-repo-advertisement/active.md.template`](templates/sub-repo-advertisement/active.md.template).

If absent: the orchestrator works without it (advertisement is
optional). If present: the orchestrator surfaces the current task /
phase / blockers in `state/sub-repos/<name>.md` and
`state/sync-status.md`.

### `<sub>/docs/decisions/` and `<sub>/docs/postmortems/`

Per-sub-repo ADRs and postmortems. Independent of the
orchestrator's `decisions/` and `incidents/` (which are macro-
level).

The orchestrator may **read** these for context. The orchestrator
generally does not write here — sub-kit owns its own
postmortems/decisions. Exceptions:

- The orchestrator may draft an ADR into `<sub>/docs/decisions/`
  if the user explicitly says "this decision belongs to the api
  repo, not the macro level." Standard PR flow.

---

## State machine for tasks

```
tasks/backlog/  →  tasks/active/  →  tasks/done/
```

Sub-kit moves task files via `git mv` as transitions occur. The
orchestrator can read all three to know what's queued, what's in
flight, and what's shipped.

---

## Branch / PR conventions in claude-kit

Per claude-kit's `task-rules.md`:

- **Sub-kit's own branches:**
  - `task/TASK-XXX-short-slug` for task work
  - `hotfix/HOTFIX-NNN-slug` for hotfixes
  - `chore/<slug>` for non-task work
- **Sub-kit's PR titles:**
  - `TASK-XXX: <task title>` for task PRs
  - Otherwise matches the branch type

When the **orchestrator** opens a PR in a kit-enabled sub-repo
(per [`sub-projects.md`](sub-projects.md) "Git management"), use:

- **Branch:** `chore/orch-<YYYY-MM-DD-short-slug>` for doc/config
  PRs; `chore/orch-task-<NNN-slug>` for task-spec drops;
  `chore/orch-install-claude-kit` for the install case
- **PR title:** `orch: <one-line summary>` — the `orch:` prefix
  disambiguates orchestrator-authored PRs from sub-kit's own work
  in the PR list

---

## Closing report (sub-kit's discipline)

Per claude-kit's `task-rules.md` "Closing report (mandatory)",
every sub-kit task PR ships with a structured closing report
covering: name, status (✅ / ⚠️ / ❌), branch, PR link, test
results (real numbers), build status, what changed, what to do
next.

The orchestrator may surface a sub-kit's recent closing report
when reading `<sub>/tasks/AUDIT.md` for context — the AUDIT entry
typically references the PR which has the closing report in its
body.

---

## Required updates to claude-kit for orchestrator v0.11.0

The orchestrator's v0.11.0 release introduces several signals
claude-kit consumes on session start. For full end-to-end
functionality, claude-kit's `task-rules.md` and related files need
the following updates. **Without these, the orchestrator side
works (writes notices, drafts inboxes), but sub-kit sessions don't
read or respond to them.**

### 1. `task-rules.md` "Active orchestrator notices" — add active-features.md

Currently the rule reads only `<sub>/.claude/active-migrations.md`
on session start. Update to read **all** files matching
`<sub>/.claude/active-*.md` (one per concern; v0.11.0 ships
migrations + features; future concerns add lazily). Render unread
or new notices to the user; surface counts in the session-start
summary.

### 2. `task-rules.md` — add shared/ channel to session-start reads

New section "Durable shared context (per-repo memory)":

> On session start, also read:
>
> - `<sub>/.claude/shared/inbox.md` — append-only repo inbox.
>   Compare entry timestamps against last-session marker (kept in
>   gitignored local-config like /inbox does for the orchestrator).
>   Surface unread entries.
> - `<sub>/.claude/shared/notes.md` — durable shared memory.
>   No unread tracking; the dev reads on demand. Mention in the
>   session-start summary that it exists.
> - `<sub>/.claude/shared/architecture.md` and
>   `<sub>/.claude/shared/references.md` — read on demand during
>   pre-flight (per the orchestrator's
>   `kit/governance/dev-preflight.md` step 5).
>
> The dev (or claude-kit's claude on the dev's behalf) may write
> back to `shared/inbox.md` to leave notes for the orchestrator
> or for future sessions. Append-only with the same shape as
> `state/inbox/` (ISO timestamp + sender + subject + body).

### 3. `task-rules.md` — reference orchestrator dev-preflight checklist

Add a pointer to the orchestrator's
`.claude/governance/dev-preflight.md` (relative path
`../../.claude/governance/dev-preflight.md` from the sub-repo) as
the canonical pre-flight discipline that applies to dev work in
sub-repos. Claude-kit's existing pre-flight discipline (if any)
becomes the platform-specific extension — not replaced, just
nested under the orchestrator's general checklist.

### 4. `task-template.md` — extended required sections

The orchestrator's `kit/governance/task-spec-shape.md` now
specifies these required body sections in every task spec
(orchestrator drafts already conform):

- User request
- Why
- **Assumptions** *(new)* — the doer's restatement
- Technical breakdown
- Acceptance criteria
- **Out of scope** *(new)*
- **Contract impact** *(new)*
- References
- **Definition of done** *(new)* — beyond AC

Update claude-kit's `task-template.md` to include all of these.
Existing sub-repo tasks remain valid (governance applies to new
specs going forward).

### 5. New file expected: `<sub>/.claude/active-features.md`

Auto-written by the orchestrator's `/feature` skill (v0.11.0).
Format spec at
[`kit/templates/sub-repo-notices/features.md.template`](templates/sub-repo-notices/features.md.template).
Same auto-managed discipline as `active-migrations.md` (regenerated
wholesale, deleted on empty).

### 6. New directory expected: `<sub>/.claude/shared/`

Created by the orchestrator's `/register` step 11 scaffold (when
the user accepts the offer). Contains four files: `architecture.md`,
`inbox.md`, `notes.md`, `references.md`. Format specs at
[`kit/templates/sub-repo-shared/`](templates/sub-repo-shared/).
Hand-edited (or appended); never auto-regenerated.

### Verification after claude-kit updates

After applying these changes in claude-kit, verify in a fresh
session:

- Sub-kit reads and surfaces both `active-migrations.md` and
  `active-features.md` if either is present.
- Sub-kit reads `shared/inbox.md` and surfaces unread entries.
- Sub-kit reads `shared/notes.md` and mentions it as available.
- Sub-kit's task-template generates new specs with all 9
  required body sections (per
  `kit/governance/task-spec-shape.md`).

---

## Updating this reference

If claude-kit's structure changes meaningfully (new files in the
kit, restructured directory layout, new skill set, changed
lifecycle), this file goes stale. Discipline:

1. Read the latest claude-kit at
   <https://github.com/ChazzCoin/claude-kit>:
   - `kit/` directory structure
   - `bootstrap/` files
   - `MANIFEST.json`
   - `kit/task-rules.md` (the operating discipline)
   - `kit/skills/*/SKILL.md` (any added or changed)
2. Update this file's "Top-level layout" + "Key files" + "Branch /
   PR conventions" sections.
3. Bump the **Last synced from claude-kit** + **Pinned to commit**
   lines at the top.
4. AUDIT entry: `🔧 claude-kit-reference.md updated to <kit-sha>.
   Changes: <one-line summary>.`

The discipline parallels how a sub-kit treats its `task-rules.md`
or `ios-conventions.md` — these are kit-managed reference files,
re-synced on demand.
