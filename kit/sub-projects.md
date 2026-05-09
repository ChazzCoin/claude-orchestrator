# Sub-projects

How this orchestrator relates to the codebases it tracks. Generic
across all instances — synced via `/sync` (kit-managed).

The orchestrator is **sub-project agnostic.** A sub-project may or
may not have claude-kit installed; the orchestrator works with
either. Functionality is richer when claude-kit is present, but the
orchestrator never *requires* it.

This file is reference material. Skills implement the behaviors
described here; this document is the contract those skills follow.

---

## Two flavors

### Kit-enabled

A sub-project is **kit-enabled** when it has
`<sub>/.claude/foundation.json` present with `kit.repo` containing
`claude-kit`. This is detected at registration and re-checked at
each `/sync-check`.

For kit-enabled projects, the orchestrator knows the file shape
(see [`claude-kit-reference.md`](claude-kit-reference.md)) and can:

- Read `CLAUDE.md`, `task-rules.md`, platform-prefix files
- Read `tasks/{backlog,active,done}/`, `tasks/PHASES.md`,
  `tasks/ROADMAP.md`, `tasks/AUDIT.md`
- Read `docs/decisions/`, `docs/postmortems/`
- Read `<sub>/.claude/active.md` (advertisement protocol — see below)
- Detect kit version from `<sub>/.claude/foundation.json` `pinned_sha`
- Draft task specs into `tasks/backlog/` (for coding work only —
  see write discipline below)
- Add phases, edit project CLAUDE.md, etc. directly via PR

### Non-kit

A sub-project without `<sub>/.claude/foundation.json` is **non-kit**.
The orchestrator gets the basic treatment:

- Read `git log`, current branch, open PRs (via `gh`)
- Read top-level `README.md` if present
- Open branches and PRs the same way as kit-enabled
- Drop migration notices into `<sub>/.claude/active-migrations.md`
  (the directory gets created if missing — the file is plain
  markdown any human can read)
- Skip everything kit-specific (no task tracking, no AUDIT log,
  no advertisement protocol read, no `task-rules.md` reference)

When the orchestrator first registers a non-kit sub-project, it
**offers** (once, not repeatedly) to install claude-kit:

> "This sub-project doesn't have claude-kit. Want me to install it?
> It would let me track tasks, see active work, and push migration
> notices the sub-kit will read on session start."

The user accepts (orchestrator runs claude-kit's `bin/init` against
the sub-project) or declines (decision is recorded in the manifest
notes; the offer is not repeated). The user can always run the
install later by hand.

---

## Read protocol

What the orchestrator reads from a sub-project, when, and from
which flavor.

| File / signal | kit-enabled | non-kit | Read when |
|---|---|---|---|
| `git log`, `gh pr list` | ✓ | ✓ | `/sync-check`, on demand |
| `git status` (uncommitted detection) | ✓ | ✓ | before any orchestrator write |
| `<sub>/.claude/foundation.json` | ✓ — kit version detection | n/a | `/register`, `/sync-check` |
| `<sub>/.claude/active.md` | ✓ — advertisement | n/a | `/sync-check` |
| `<sub>/.claude/active-migrations.md` | ✓ — confirm what we wrote | ✓ — same | optional |
| `<sub>/CLAUDE.md` | ✓ — confirm orchestrator back-pointer | ✓ — context only | `/register`, on demand |
| `<sub>/tasks/{backlog,active,done}/` | ✓ — task state | n/a | `/sync-check`, on demand |
| `<sub>/tasks/PHASES.md`, `tasks/ROADMAP.md` | ✓ | n/a | on demand |
| `<sub>/tasks/AUDIT.md` | ✓ — sub-kit's chronological log | n/a | on demand |
| `<sub>/docs/decisions/`, `<sub>/docs/postmortems/` | ✓ | ✓ if present | on demand |
| `<sub>/README.md` | ✓ — context | ✓ — context | on demand |

---

## Read-then-act protocol

When the user asks the orchestrator to do something involving a
sub-project, the routing is:

1. **Read the sub-project's docs first.** `<sub>/docs/` (runbooks,
   ADRs, postmortems), `<sub>/CLAUDE.md`, kit's `task-rules.md` if
   kit-enabled, conventions, READMEs — whatever's relevant. The
   sub-project is the source of truth for its own how-to. The
   orchestrator's job is to USE that knowledge, not duplicate it.
2. **Classify the action:**

   | Action type | What it is | What the orchestrator does |
   |---|---|---|
   | **Read query** | Command that queries external state without changing it (`gh pr list`, `az resource list`, `kubectl get`, `git log`, etc.) | Run it directly |
   | **State change via documented command** | Command from a sub-project runbook that changes external state (deploy, scale, rotate, restart) | Run **with explicit user approval** per CLAUDE.md "Executing actions with care" |
   | **Non-running file update** | Edit a doc (README, runbook, ADR, conventions, ROADMAP, PHASES, AUDIT, CLAUDE.md, etc.) | Draft in `chore/orch-*` PR; user merges |
   | **Code change** *(running files — see boundary below)* | Modify a file that gets built / interpreted / deployed / executed | **Orchestrator does NOT do this** without explicit user override (see "Code boundary"). File a task spec via path 3, or advise the user to do it manually. |

3. **Act.** Take the matching path.

The principle: the orchestrator is a **competent operator with read
access to everything and write access only to docs**. It runs the
sub-project's own playbooks rather than inventing new ones.

---

## Running files vs non-running files

The boundary that determines whether the orchestrator can write a
file directly.

### Running files *(orchestrator does NOT write — see Code boundary)*

Anything that gets built, interpreted, deployed, or executed in a
running environment:

- Source code (any language)
- Build / packaging configs (`package.json`, `Cargo.toml`,
  `build.gradle`, `Package.swift`, `pyproject.toml`, etc.)
- CI / CD pipeline files (`.github/workflows/*.yml`,
  `Jenkinsfile`, etc.)
- Infrastructure-as-code (`*.tf`, `*.bicep`, `Pulumi.*.yaml`,
  `cloudformation.*`, etc.)
- Runtime configs loaded by services (`.env*`, `config/*.{json,yaml,toml}`,
  Kubernetes manifests, etc.)
- Schema migration scripts (the executable kind)
- Generated code, vendored dependencies

**The orchestrator READS these freely** — to understand behavior,
verify what's actually deployed, inform decisions. It never writes
them without explicit override.

### Non-running files *(orchestrator can read AND write via PR)*

Files that don't get loaded or executed at runtime:

- Markdown docs (`README.md`, `docs/runbooks/`, `docs/postmortems/`,
  `docs/decisions/`, `<sub>/CLAUDE.md`, `task-rules.md`,
  `<sub>/tasks/PHASES.md`, `<sub>/tasks/ROADMAP.md`,
  `<sub>/tasks/AUDIT.md`)
- Orchestrator-pushed notices (`<sub>/.claude/active-*.md`)
- Conventions docs, design docs, requirement specs
- Static documentation that lives next to code but isn't loaded by
  it

**Edge cases:**

- **Comments / inline docs inside running files** — these are part
  of the running file. Off-limits without override. Doc-only edits
  to a source file go through the task pipeline.
- **`.gitignore`, `.editorconfig`, etc.** — config files for tools
  that don't run in production. The orchestrator can update these
  via PR.
- **Test fixtures / mock data** — running files (loaded by tests).
  Off-limits without override.

**The test:** *"Does anything that runs in production load this
file?"* Yes → code. No → doc.

---

## Write protocol

The orchestrator writes into sub-projects in four cases. The
discipline split is the load-bearing rule.

### 1. Migration notices — `<sub>/.claude/active-migrations.md`

The structured cross-repo coordination signal. Owned by
`/migration`. Template at
[`kit/templates/sub-repo-notices/migrations.md.template`](templates/sub-repo-notices/migrations.md.template).
Works on kit-enabled and non-kit alike.

### 2. Non-running file updates — orchestrator touches files directly via PR

For non-running files only (see "Running files vs non-running files"
above). Examples: schema documentation, phase addition, ROADMAP
edit, AUDIT entry, project `CLAUDE.md` correction, ADR drafted
into `<sub>/docs/decisions/`, runbook update, conventions doc fix.

The orchestrator:

1. Pulls latest main in the sub-repo
2. Branches: `chore/orch-<YYYY-MM-DD-short-slug>`
3. Drafts the file change directly
4. Opens a PR with title `orch: <one-line summary>` and a body
   linking back to the orchestrator artifact (ADR, migration,
   feature) that motivated the change
5. **Lets the user approve the merge**

**No task spec is created for non-running file updates.** Doc edits
are not "work" in the sub-kit sense — they're reflection of
decisions already made at the macro level.

This rule is the load-bearing distinction:

> **Tasks are for coding work. Non-running file edits the
> orchestrator does itself via PR. Running files are off-limits
> without explicit user override.**

### 3. Coding changes — task spec drafted into `<sub>/tasks/backlog/`

When the user decides a code change is needed (bug fix, feature
implementation, refactor) in a kit-enabled sub-project, the
orchestrator:

1. Drafts a task spec into `<sub>/tasks/backlog/TASK-NNN-slug.md`
   per claude-kit's `task-template.md` shape (see
   [`claude-kit-reference.md`](claude-kit-reference.md))
2. Optionally adds the task line to `<sub>/tasks/ROADMAP.md` under
   the appropriate phase
3. Opens a PR (`chore/orch-task-NNN-slug`) so the user can review
   and merge the spec
4. The sub-kit's claude-kit instance picks up the task on next
   session and works it

**The orchestrator never executes the code work.** It files the
spec; the sub-kit ships it.

For non-kit sub-projects: the orchestrator can't draft a task spec
(there's no `tasks/` system). Either the user installs claude-kit
first, or the work happens outside the orchestrator's coordination
(an issue / GitHub project / chat message — orchestrator can
optionally draft the GitHub issue and link it).

### 4. claude-kit installation — orchestrator runs `bin/init`

If the user accepts the install offer, the orchestrator runs
`<claude-kit>/bin/init <sub-project-path>`. This is a single
sanctioned write that creates the kit shape in the target.
Standard PR flow afterward (the init creates files; orchestrator
opens a PR with all of them so the user can review).

---

## Code boundary (non-negotiable)

The orchestrator does **NOT** modify running files (per the list
above) without **explicit user override**. This rule is hard.

### Why the rule exists

The orchestrator doesn't have the verification gate, the
test-pairing discipline, or the per-repo context that the sub-kit
has. Code changes made by the orchestrator skip the very controls
that make sub-kit-shipped code reliable. Hand-edits to running
files are legitimately risky — the orchestrator is not where code
work happens.

Doc edits don't have that risk: there's no runtime that fails
silently on a misworded README.

### What to do when the user asks for a code change

When the user asks the orchestrator to modify a running file, the
orchestrator must:

1. **Pause** before acting.
2. **Identify it as a code touch** clearly:
   > "The change you're asking for involves `<file>`, which is a
   > running file (it gets [built / loaded / executed in production]).
   > By default the orchestrator doesn't touch code."
3. **Advise against it.** Lay out the reasoning:
   - Code changes go through the sub-kit's task pipeline
     (verification gate, focused test, closing report) — that
     pipeline doesn't apply when the orchestrator hand-edits.
   - The orchestrator lacks per-repo context (build config,
     test fixtures, runtime quirks) that the sub-kit has.
   - Even small code changes can break things in non-obvious
     ways without the gate.
4. **Suggest the right path:**
   - **Kit-enabled sub-project:** file a task spec via path 3
     (orchestrator drafts the spec into `<sub>/tasks/backlog/`,
     sub-kit ships it).
   - **Non-kit sub-project:** offer to install claude-kit; or the
     user does the change manually.
5. **Wait for explicit override.** If the user says (paraphrase
   acceptable):
   - "Yes, override — do it anyway"
   - "Yes, I'm overriding the rule"
   - "Override the code boundary"
   - "Touch the code"
   …then the orchestrator complies. Anything ambiguous → ask
   again, don't proceed.

### When the user overrides

If the user explicitly overrides:

- **Open the PR through the standard `chore/orch-*` flow.**
- **PR body must contain an explicit override marker:**
  > **⚠ Code touch under user override.** Standard sub-kit
  > verification gate not applied. The user requested this
  > orchestrator-direct edit on `<YYYY-MM-DD>`.
  > Recommended: run `<sub-repo's verification command per
  > CLAUDE.md>` before merging.
- **AUDIT entry has the explicit override flag (use 🚨):**
  > `🚨 Code touch under override — <repo> <file> <one-line>.
  > User overrode code boundary. → PR #<N>`
- **User still approves the merge.** Override doesn't bypass the
  PR flow; it bypasses only the "we don't touch code" default.
- **Sub-kit's per-repo `/review` should run on the PR.** The
  override skips the orchestrator's default rule, not the sub-
  kit's normal review pipeline.

### What "code change" means in practice

When unsure, ask: *"Does anything that runs in production load
this file?"* If yes, treat as code. Examples:

- Editing `index.js`, `main.swift`, `models.py` → code
- Editing `README.md` → not code
- Editing `package.json` (changes what the runtime depends on) → code
- Editing `docs/runbooks/deploy.md` → not code
- Editing `.github/workflows/ci.yml` (CI runs this) → code
- Editing `tasks/ROADMAP.md` (the orchestrator nor sub-kit runs
  this) → not code
- Editing a comment inside `index.js` → code (the file is
  running; the comment is in it)
- Adding a new doc file at `docs/architecture.md` → not code

### What the orchestrator does NOT have to advise against

- Running commands sourced from sub-project docs (read queries:
  always; state changes: with user approval per CLAUDE.md).
- Reading running files for context.
- Updating non-running files.
- Filing task specs (path 3) into kit-enabled sub-projects.

These don't require the override — they're the default protocol.

---

## Sub-kit advertisement protocol

A sub-kit MAY write `<sub>/.claude/active.md` to advertise its
current state to the orchestrator. **Voluntary** — the orchestrator
works with or without.

Format (see
[`templates/sub-repo-advertisement/active.md.template`](templates/sub-repo-advertisement/active.md.template)):

- Current task ID + title (or "idle")
- Current phase from `tasks/PHASES.md`
- Current branch
- ETA / blockers (if known)
- Notes for the orchestrator (free-form, optional)

The orchestrator reads this on `/sync-check` and surfaces it in
`state/sync-status.md` and `state/sub-repos/<name>.md`.

When this protocol matures (see "Future direction" below), the
sub-kit can read the orchestrator's state to validate plans against
its own work — bidirectional context flow.

---

## Git management

The orchestrator has direct git authority over registered
sub-repos when making changes. Discipline:

### Fetching

- **`/refresh` is the canonical fetch.** Iterates the manifest,
  runs `git fetch origin` against every cloned sub-repo, records
  per-repo ahead/behind + dirty flags, writes `state/last-fetch.json`
  with a timestamp. Read-only against remotes; never auto-pulls.
- **Session-start staleness check.** Per
  `orchestrator-rules.md`, the orchestrator checks
  `state/last-fetch.json` on session start. >24h or missing → warn
  and suggest `/refresh` before compiler skills run.
- **Before any orchestrator write** — fetch the target sub-repo +
  verify a clean working tree. If dirty, abort the write with:
  "<sub-repo> has uncommitted changes; commit, stash, or revert
  before I make orchestrator changes there." User unblocks;
  orchestrator retries.
- **Pulls are not orchestrator work.** If a sub-repo is behind
  `origin/main`, the user pulls it themselves. The orchestrator
  surfaces the drift but doesn't fast-forward working trees.

### Branching

- **Always branch from up-to-date main.** Pull first, then branch.
- **Branch naming:** `chore/orch-<YYYY-MM-DD-short-slug>` for
  documentation / config / spec PRs; `chore/orch-task-<NNN-slug>`
  for task-spec drops; `chore/orch-install-claude-kit` for the
  install case.
- **Never write to main directly.** No exceptions. Even if the
  change is "trivial" — the PR is the discipline.

### PRs

- **Title:** `orch: <one-line summary>` so the sub-repo's PR list
  shows clearly which came from the orchestrator vs. the sub-kit's
  own work.
- **Body:** link back to the orchestrator artifact (ADR ID,
  migration ID, feature ID, AUDIT entry timestamp) that motivated
  the change. Reviewer should be able to follow the link to
  understand the why.
- **Never auto-merge.** The user always approves the merge. This is
  non-negotiable, including for "trivial" doc PRs.
- **Don't push hooks (`--no-verify`)** unless the user explicitly
  asks. Honor any branch protection rules.

### Pull strategy on rebase

Default: `git pull --rebase` for orchestrator branches against
their base. Defer to the project's preference if `<sub>/CLAUDE.md`
specifies one.

### Rollback

If a merged orchestrator PR turns out wrong, the rollback is a new
PR (`git revert`), not a force-push. Same approval flow.

### Reviewing PRs (orchestrator-grade)

Separate from sub-kit's `/review` (which catches line-level issues).
The orchestrator's review checks **macro fit** — does the PR honor
the contracts, conventions, principles, and active migrations
recorded here?

**Trigger conditions** *(not every PR — only when one applies):*

- The PR touches a sub-repo affected by an active migration
- The PR touches a contract surface (`contracts/*.md` domains)
- The PR touches conventions (auth, naming, error handling), vendors,
  or schemas
- The PR was authored by the orchestrator itself (`chore/orch-*`)
- The user explicitly asks

**What the review reads:** PR diff + spec, plus orchestrator
artifacts — `contracts/`, `conventions/`, `platform-constraints.md`,
`tech-principles.md`, the affected migration's per-repo plan,
recent incidents.

**Outputs:** filed at `pr-reviews/YYYY-MM-DD-<repo>-pr<N>.md`; PR
comment posted via `gh pr comment` (with user approval); spawned
artifacts (risks, ADRs) when applicable. Three statuses: ✅ approved,
⚠ conditional, ❌ request changes.

**Never auto-approve, never auto-merge.** The user reviews the
review.

Full discipline at [`pr-reviews/README.md`](pr-reviews/README.md).
The two reviews compose: sub-kit catches line-level, orchestrator
catches macro fit.

---

## Tracking

Each registered sub-repo has a state file at
`state/sub-repos/<name>.md` holding the orchestrator's accumulated
view of it: kit-enabled flag, last sync, advertisement, references
to all migrations / features / ADRs / risks / incidents touching
it. See [`state/README.md`](state/README.md) for the full schema.

The manifest at `state/manifest.md` is the **index** (one entry per
sub-repo, minimal metadata). Per-sub-repo state files are the
**depth** (one file per sub-repo, accumulated over time).

`/sync-check` updates the auto-tracked fields (HEAD, last sync,
advertisement). Manual notes in the per-sub-repo file's **Notes**
section are preserved.

## Registration

How sub-repos physically land on disk relative to the orchestrator
is documented in detail at
[`sub-project-registration.md`](sub-project-registration.md).

Headline rules:

- **The orchestrator owns its checkouts.** Sub-repos clone into
  `<orchestrator-root>/repos/<name>/` — convention, not
  configuration. Path is identical on every machine. `repos/` is
  gitignored.
- **Two flows, two lifecycles.** `/register` (per-repo, deliberate,
  first-time setup) drafts a manifest entry and clones the new
  sub-repo. `bin/setup` (bulk, automatic, collaborator onboarding)
  reads the manifest and clones any sub-repos not yet present.
- **Existing checkouts elsewhere are not referenced.** If the user
  has the same repo cloned at `~/work/<name>/`, that's a personal
  working copy — opaque to the orchestrator. The orchestrator's
  `repos/<name>/` is the canonical local view.
- **Mobile / remote-only fallback.** Read-side skills query `gh api`
  against the manifest's `git_remote` when no local clone is
  present. Write-side operations require a local clone (run
  `bin/setup` first).
- **Never registers without `git_remote`.** A repo with no remote
  can't be coordinated — refuse and ask the user to add one.

---

## When NOT to use the orchestrator's sub-repo machinery

- **For pure code work** — the sub-kit does that directly. The
  orchestrator only files specs and reads results.
- **For per-repo postmortems** — those live in the sub-repo's
  `docs/postmortems/`. Only lift to `<orchestrator>/incidents/`
  when the lessons span repos (see `incidents/README.md`).
- **For per-repo decisions** — those live in the sub-repo's
  `docs/decisions/`. Only lift to `<orchestrator>/decisions/` when
  the call has cross-repo implications.

---

## Future direction *(not yet wired)*

claude-kit will eventually grow the inverse — sub-kits read their
parent orchestrator's docs (vision, principles, contracts,
conventions, active features, active migrations) when planning
work, so the sub-kit can validate its plans against the macro view
without the user having to courier context manually.

This requires claude-kit to know how to find its parent
orchestrator (already wired via the "Macro architecture" section
in `bootstrap/CLAUDE.md.template`) and to know what to read there.
The reading shape will need a sub-kit-side analog of the
orchestrator's `claude-kit-reference.md` — call it
`orchestrator-reference.md` shipped in claude-kit's `kit/`.

When that lands, the orchestrator's `claude-kit-reference.md` will
note the bidirectional protocol.
