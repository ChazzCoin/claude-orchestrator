# Sub-project registration

How a sub-repo gets associated with the orchestrator and how its
checkout lands on disk. The discipline `/register` and `bin/setup`
follow. Generic across all instances — synced via `/sync`
(kit-managed).

## The principle

**The orchestrator owns its checkouts.** Sub-repos clone into
`<orchestrator-root>/repos/<name>/` — a fixed convention path. The
orchestrator does not reference external clones on the user's
machine, does not move or relocate existing repos, and does not
maintain per-user / per-machine path configuration.

Path is **convention, not configuration**:

- Always `<orchestrator-root>/repos/<name>/`
- Never stored in any state file
- Never varies per machine
- Computed from the orchestrator's own location plus the manifest
  entry name

`repos/` is gitignored from the orchestrator's git tree. Each
sub-repo retains its own `.git` and behaves normally inside its
directory.

If a user has existing checkouts of these repos elsewhere on their
machine, the orchestrator does not reference them. The
orchestrator-managed clones under `repos/` are the canonical local
copies for orchestrator-driven operations. Users may keep their
other checkouts as personal working copies, but those are opaque to
the orchestrator.

## Layout

```
<company>-orchestrator/             cloned from <org>/<company>-orchestrator
├── .git/                            the orchestrator's repo
├── .claude/                         skills, rules, foundation.json
├── bin/setup                        bulk-clones all registered sub-repos
├── decisions/, state/, ...          the brain
└── repos/                           GITIGNORED
    ├── api/                         (its own .git, working copy)
    ├── ios/
    ├── web/
    └── devops/
```

## Two flows

A sub-repo lands at `repos/<name>/` via one of two flows:

### `/register` — first-time, per-repo, deliberate

Used when a company is bootstrapping their orchestrator and adding a
new sub-repo to the manifest for the first time. Per-repo,
interactive, deliberate.

Process (full contract in `kit/skills/register/SKILL.md`):

1. User provides:
   - **Git remote URL** (e.g. `git@github.com:<org>/<repo>.git`)
   - **Role** (api / ios / web / devops / other — one word)
   - **Name** (short identifier — manifest section heading and
     directory name under `repos/`)

2. Orchestrator verifies:
   - `git ls-remote <url>` succeeds
   - No existing manifest entry has the same `name` or `git_remote`
   - `repos/<name>/` either doesn't exist OR exists with a matching
     remote

3. Orchestrator drafts the manifest entry, shows it, gets user
   confirmation.

4. On confirmation:
   - Append to `state/manifest.md`
   - Create `state/sub-repos/<name>.md` from the template
   - `git clone <url> repos/<name>` (best-effort — clone failures
     don't roll back the manifest entry; the user can retry via
     `bin/setup`)
   - Suggest a `## Macro architecture` block for the sub-repo's
     `CLAUDE.md`

### `bin/setup` — bulk, automatic, collaborator onboarding

Used when a collaborator (or the user themselves on a new machine)
has just cloned the orchestrator instance and needs `repos/`
populated locally. Fully automatic, no prompts.

Process:

1. Read `state/manifest.md`.
2. For each registered entry, check `repos/<name>/`:
   - Doesn't exist → clone from `git_remote`
   - Exists with matching remote → skip
   - Exists with different remote → flag as collision, skip
3. Report per-repo outcomes at the end (cloned / skipped /
   collision / failed).
4. Idempotent — re-runnable to recover from per-repo failures.

See `bin/setup` for the script.

## Mobile / remote-only mode

Mobile devices and other environments without local filesystem
access don't run `bin/setup` and don't populate `repos/`. Read-side
skills (`/sync-check`, `/status`, `/brief`) fall back to `gh api`
queries against the manifest's `git_remote` for branch state, HEAD
SHA, and sub-kit advertisements (via the contents API).

Write operations — `/migration` updating
`<sub-repo>/.claude/active-migrations.md`, `/register` cloning a new
sub-repo — require a local clone under `repos/<name>/`. From mobile,
those either route through `gh api`-driven PR flows or get deferred
to a machine that has run `bin/setup`.

## What the orchestrator never does

- **Never references external checkouts.** The orchestrator does not
  look outside `<orchestrator-root>/repos/` for sub-repo state. Other
  clones on the user's machine are personal working copies, not
  orchestrator-managed.
- **Never deletes a `repos/<name>/` checkout automatically.** The
  orchestrator clones; cleanup is the user's call.
- **Never auto-clones without an explicit trigger.** `/register` and
  `bin/setup` both clone, but only when the user invokes them.
- **Never registers without `git_remote`.** A repo with no remote
  can't be coordinated with from the orchestrator (no `gh pr list`,
  no back-pointer for sub-kit integration). Refuse and ask the user
  to add a remote.

## Edge cases

### `repos/<name>/` already cloned with the right remote

Idempotent — `bin/setup` and `/register` both detect this and skip
the clone. The existing checkout is preserved with whatever local
state, branches, or uncommitted changes the user has.

### `repos/<name>/` already cloned with a different remote

Name collision. Surface to the user. They rename or remove the
existing directory; the orchestrator doesn't auto-resolve.

### Repo doesn't exist on GitHub yet

Out of scope for `/register`. Create the GitHub repo first
(`gh repo create`), then `/register` it.

### Repo shared between two company orchestrators

Each orchestrator owns its own checkout. Two companies that share a
library register it independently — each gets a separate clone in
its own `repos/<name>/`. The library's `CLAUDE.md` may mention both
orchestrators under "Macro architecture" if the sub-kit needs to
read from both.

### User has the same repo cloned in multiple places

If the user has the same repo at `repos/<name>/`
(orchestrator-managed) and at `~/work/<name>/` (personal IDE
checkout), the orchestrator only sees `repos/<name>/`. The user's
personal checkout is fine to keep but is not orchestrator-visible.

## Anti-patterns

- **"Let me register the path to my existing checkout."** Not the
  flow. Provide the `git_remote`; the orchestrator clones fresh
  into `repos/<name>/`. Existing checkouts elsewhere are abandoned
  by the orchestrator (kept by the user as personal copies, if they
  want).
- **"Let me symlink my existing checkout into `repos/<name>/`."**
  Don't. The point of the convention is uniformity across machines.
  A symlink layer adds fragility and reintroduces the per-machine
  variance the design was meant to eliminate.
- **"Let me commit `repos/<name>/` to the orchestrator's git tree
  for portability."** No. `repos/` is gitignored. Sub-repos are
  separate git histories — committing them as nested content is
  exactly the failure mode this design avoids.
- **"Let me skip the `git_remote` requirement for this internal
  repo."** Don't. Without a remote, the orchestrator can't
  coordinate (no `gh pr list`, no PR flows, no remote-only
  fallback). Add a remote first — even a private GitLab / Gitea
  works.
