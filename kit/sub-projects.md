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

## Write protocol

The orchestrator writes into sub-projects in four cases. The
discipline split is the load-bearing rule.

### 1. Migration notices — `<sub>/.claude/active-migrations.md`

The structured cross-repo coordination signal. Owned by
`/migration`. Template at
[`kit/templates/sub-repo-notices/migrations.md.template`](templates/sub-repo-notices/migrations.md.template).
Works on kit-enabled and non-kit alike.

### 2. Documentation / config / spec changes — orchestrator touches files directly via PR

When the user decides on a non-coding change — schema documentation,
phase addition, ROADMAP edit, AUDIT entry, project CLAUDE.md
correction, ADR drafted into `<sub>/docs/decisions/`, etc. — the
orchestrator:

1. Pulls latest main in the sub-repo
2. Branches: `chore/orch-<YYYY-MM-DD-short-slug>`
3. Drafts the file change directly
4. Opens a PR with title `orch: <one-line summary>` and a body
   linking back to the orchestrator artifact (ADR, migration,
   feature) that motivated the change
5. **Lets the user approve the merge**

**No task spec is created for documentation updates.** Doc edits
are not "work" in the sub-kit sense — they're reflection of
decisions already made at the macro level.

This rule is the load-bearing distinction:

> **Tasks are for coding work. Documentation / config / spec
> changes the orchestrator does itself.**

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

### Pulling

- **On `/sync-check`** — for each registered sub-repo, `git fetch`.
  If `git status` shows uncommitted changes, **don't pull** — note
  it and continue. Compare local HEAD to upstream HEAD; record in
  the per-sub-repo state file.
- **Before any orchestrator write** — fetch + verify clean working
  tree. If dirty, abort the write with: "<sub-repo> has uncommitted
  changes; commit, stash, or revert before I make orchestrator
  changes there." User unblocks; orchestrator retries.

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
