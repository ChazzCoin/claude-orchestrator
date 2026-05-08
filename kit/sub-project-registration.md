# Sub-project registration

How a sub-repo physically gets associated with the orchestrator.
The discipline that `/register` follows. Generic across all
instances — synced via `/sync` (kit-managed).

## The principle

**Keep repos where they live.** The orchestrator records the
absolute local path; it doesn't move, clone, or symlink existing
repos.

This is option 1 of four considered:

| Option | Decision |
|---|---|
| 1. Keep where they live, record absolute paths | **Chosen.** |
| 2. Clone into orchestrator dir (`<orchestrator>/repos/api/`) | Rejected — duplicates checkouts, "which is canonical" confusion. |
| 3. Symlink (`<orchestrator>/repos/api → ~/.../api`) | Rejected — adds a layer for marginal benefit. |
| 4. Remote-only (clone to temp cache per op) | Future upgrade path when multi-machine becomes pressing; not v1. |

The manifest's `git_remote` field is the portable anchor — when /
if option 4 becomes the right move, the remote is what survives the
migration.

## Recommended layout

The orchestrator and its registered sub-repos live as **siblings**:

```
~/ChazzCoin/                        (or wherever your code lives)
├── <company>-orchestrator/         this orchestrator instance
├── <company>-api/                  registered sub-repo
├── <company>-ios/                  registered sub-repo
├── <company>-web/                  registered sub-repo
└── <company>-devops/               registered sub-repo
```

Why sibling layout:

- **Predictable defaults.** When `/register` clones a new sub-repo
  from GitHub, default destination is `<orchestrator-parent>/<name>`
  — the sibling slot.
- **Easy enumeration.** `ls ~/ChazzCoin/<company>-*` shows the
  whole stack at a glance without needing the orchestrator open.
- **No nesting confusion.** The orchestrator isn't a parent of
  sub-repos; nothing should imply hierarchy where there isn't one.

The convention is recommended, not enforced. The manifest stores
absolute paths, so the orchestrator works fine if a repo lives
somewhere unconventional. But for a fresh stack, sibling layout
is the default.

## Registration input cases

When the user says "register the api repo," `/register` walks
through three input cases:

### Case 1 — Path exists, repo is local

User: "register `~/ChazzCoin/vsi-api`"

Orchestrator validates:

1. Path exists
2. Path is a git repo (`.git/` directory or git worktree)
3. Has a remote `origin`
4. Reads `git remote get-url origin` for the canonical URL
5. Reads `git symbolic-ref refs/remotes/origin/HEAD` for default branch
6. Detects kit-enabled (presence of `<path>/.claude/foundation.json`
   with `claude-kit` in `kit.repo`)

Then creates the manifest entry + per-sub-repo state file. Done.

### Case 2 — GitHub URL, repo not yet local

User: "register `github.com/ChazzCoin/vsi-api`" (or pastes the URL)

Orchestrator:

1. Suggests a clone destination — default is
   `<orchestrator-parent>/<repo-name>` (sibling layout).
   Example: orchestrator at `~/ChazzCoin/vsi-orchestrator/` →
   default clone to `~/ChazzCoin/vsi-api/`.
2. Asks the user to confirm or override the destination.
3. Runs `git clone <url> <destination>` (after explicit approval —
   this is a state-changing command per CLAUDE.md "Executing
   actions with care").
4. Falls through to Case 1 against the new local path.

### Case 3 — Neither path nor URL given

User: "register the api repo"

Orchestrator: ask. Don't guess. Don't go searching the filesystem.
The user states the location.

## What `/register` never does

- **Never moves an existing repo.** Tidying up directory layout is
  the user's call; the orchestrator adapts to where things are.
- **Never deletes an existing local checkout.** If the user already
  has the repo cloned somewhere, that path is the registration —
  no clone happens.
- **Never auto-clones without user approval.** Clones are
  state-changing; user confirms before any `git clone`.
- **Never registers without `origin`.** A repo with no remote can't
  be coordinated with from the orchestrator (no `gh pr list`, no
  back-pointer for sub-kit integration). Tell the user; ask them
  to add a remote before registering.
- **Never registers a path that's not a git repo.** Plain
  directories aren't sub-repos; the orchestrator doesn't track
  them.

## Edge cases

### Repo cloned in multiple places

If the user has `~/ChazzCoin/vsi-api/` and `~/work/vsi-api/`
(separate clones of the same repo), only one gets registered.
Pick the one that's the active working checkout. The other is
fine to keep around but isn't the orchestrator's reference.

### Git worktrees

Worktrees (created via `git worktree add`) are valid registration
targets — they have `.git`-ish state, a remote, a branch. Register
the worktree path if that's where the user actually works.

### Repo shared between two orchestrators

If two companies you're CTO of share a library (e.g., a common
auth helper), both orchestrators can register the same path.
Each manifest entry is independent. The library has two
"upstream" orchestrators — its own CLAUDE.md should mention both
under "Macro architecture" if you want the sub-kit to read both.

### Repo on a network drive / shared volume

Path-based registration still works. But: the orchestrator needs
the path to be reachable when it runs. If the volume isn't
mounted, the orchestrator should fail loudly on `/sync-check`
rather than silently skip the sub-repo.

### Fresh repo (doesn't exist anywhere yet)

Out of scope for `/register`. If the user wants to start a new
sub-project, that's a separate flow:

1. Create the GitHub repo (`gh repo create`)
2. Clone locally
3. Optionally run claude-kit's `bin/init` to make it kit-enabled
4. Then `/register` it (Case 1)

## Future-proofing — when to migrate to option 4

The "remote-only" model (manifest stores `git_remote`, orchestrator
clones to a temp cache per op) becomes the right answer when:

- **Multi-machine.** You work from a laptop and a desktop with
  different paths to the same repos.
- **Multi-person.** A team shares the orchestrator and absolute
  paths differ per person.
- **Cloud / remote orchestrator.** The orchestrator runs
  somewhere that doesn't have direct filesystem access to your
  laptop.

When that day comes:

1. Manifest entries already have `git_remote` — that's the anchor
   that survives.
2. `/sync-check` and friends switch from `git -C <path>` to
   `git fetch <url>` against a cache directory.
3. `local_path` becomes optional (still usable when present, e.g.
   for users who want to work in a checkout for IDE access).

The current option-1 design is forward-compatible with this
migration — no data loss, no manifest rewrite needed.

## Anti-patterns

- **"Let me just symlink everything into the orchestrator."** The
  symlink doesn't do work; the absolute path in the manifest does.
  Adding symlinks adds a layer with no benefit.
- **"Let me move my existing repos to follow the sibling
  convention."** Don't. Other tooling (IDE configs, shell aliases,
  scripts) likely depends on current paths. Adapt the orchestrator
  to where things are; don't relocate to please the orchestrator.
- **"Let me clone the same repo again into the orchestrator dir
  for safety."** Two checkouts of the same repo will drift. One
  registered checkout, period.
- **"Let me skip the `origin` requirement for this internal
  repo."** Don't. Without a remote, the orchestrator can't do PR
  flows or migration coordination. Either give it a remote (even
  to a private GitLab / Gitea) or accept that the orchestrator
  won't be able to coordinate with it.
