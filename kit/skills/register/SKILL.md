---
name: register
description: Register a new sub-repo with the orchestrator — adds it to state/manifest.md, creates its state file, and clones it into repos/<name>/. Triggered by "/register", "register a sub-repo", "add the api repo", "new sub-kit", "track this repo".
---

# /register — Register a sub-repo with the orchestrator

Add a new sub-repo to `state/manifest.md`, create its state file
under `state/sub-repos/<name>.md`, and clone it into the
orchestrator's workspace at `repos/<name>/`. After registration the
sub-repo is included in `/sync-check`, eligible to receive migration
notices, and visible in `/status`.

**Output pattern:** [Pattern 34 — Selection prompt](../../output-catalogue.md#34--selection-prompt)
shape for the per-input Q&A flow (one input per prompt, defaults
where reasonable, confirm before write) +
[Pattern 1 — Hero completion card](../../output-catalogue.md#1--hero-completion-card)
on successful registration + clone.

This is the **first-time setup** flow — per-repo, deliberate, used
when a company is bootstrapping their orchestrator. For onboarding a
collaborator onto an already-configured instance (clone everything
already in the manifest in one shot), use `bin/setup` instead.

## Behavior contract

- **Verify the git remote is reachable.** `git ls-remote <url>` —
  if it fails, refuse and explain. We don't proceed until the URL is
  valid.
- **Verify it's not already registered.** If a sub-repo with the
  same name or `git_remote` is in the manifest, point that out.
- **Don't trust user-typed URLs without verification.** A typo or
  stale URL becomes a permanent manifest entry otherwise.
- **Don't update the sub-repo's `CLAUDE.md`.** Recommend the
  addition to the user; registering is a one-way orchestrator-side
  action.
- **Best-effort clone.** If `git clone` into `repos/<name>` fails
  (auth, network), keep the manifest entry — the clone is recoverable
  by re-running `bin/setup`. The manifest is the truth, the clone is
  a side effect.
- **Path is convention, not configuration.** Sub-repos always live at
  `repos/<name>/` relative to the orchestrator root. Don't ask the
  user for a path.

## Process

1. Ask the user for:
   - **Git remote URL** (e.g. `git@github.com:<org>/<repo>.git`)
   - **Role** (api / ios / web / devops / other — one word)
   - **Name** (short identifier — the manifest section heading and
     the directory name under `repos/`. Suggested default: the role,
     or the basename of the URL minus `.git`.)

2. Verify the remote:
   - `git ls-remote <url>` succeeds (auth + URL valid)
   - Get default branch via `git ls-remote --symref <url> HEAD` —
     parse the symref line for `refs/heads/<branch>`

3. Verify the manifest:
   - No existing entry has the same `name` heading
   - No existing entry has the same `git_remote`

4. Verify the local workspace:
   - `repos/<name>/` does not already exist, OR if it does, its
     `git -C repos/<name> config --get remote.origin.url` matches
     the new `git_remote` (in which case skip the clone in step 9)

5. Draft the new manifest entry:

   ```markdown
   ## <name>
   - **Role:** <role>
   - **Git remote:** <url>
   - **Default branch:** <main / master / etc>
   - **Kit-enabled:** unknown
   - **Registered:** <YYYY-MM-DD>
   - **Last synced HEAD:** never
   - **Last synced at:** never
   - **State file:** [state/sub-repos/<name>.md](state/sub-repos/<name>.md)
   - **Notes:**
   ```

6. Show to the user. Confirm.

7. Append to `state/manifest.md` under the `## Sub-repos` heading
   (alphabetical order recommended).

8. Create `state/sub-repos/<name>.md` from
   `state/sub-repos/_template.md` — populate the frontmatter
   (`name`, `role`, `git_remote`, `default_branch`, `registered`,
   `kit_enabled: unknown`, `kit_version: n/a`,
   `last_synced_head: never`, `last_synced_at: never`,
   `last_pulled_at: never`, `kit_install_offered: n/a`,
   `kit_install_declined: n/a`). Leave the body sections to be
   filled by `/sync-check`.

9. Clone into `repos/<name>/`:

   ```sh
   git clone <git_remote> repos/<name>
   ```

   If this fails, surface the error and tell the user they can retry
   later via `bin/setup`. Do not roll back the manifest entry —
   manifest is the source of truth, clone failures are recoverable.

10. Suggest the user add this to the sub-repo's `CLAUDE.md` (don't
    write it for them):

    ```markdown
    ## Macro architecture
    The cross-repo architectural truth lives in the company's
    orchestrator (private repo). Read `stack/inventory.md`,
    `contracts/`, `conventions/`, and any `features/<x>.md` matching
    current work before making cross-platform decisions. When this
    sub-repo is checked out under the orchestrator's workspace, the
    orchestrator root is `../../`.

    When working on a task, also check `.claude/active-migrations.md`
    in this repo — it lists open cross-repo migrations affecting this
    codebase.
    ```

## Style rules

- **One sub-repo per call.** Don't batch.
- **Confirm before writing.** Show the manifest entry and the clone
  destination before executing.
- **Honest about what registration does and doesn't do.** It adds a
  manifest entry, creates a state file, and clones the repo. It does
  not modify anything inside the cloned repo.

## What you must NOT do

- **Don't write into the sub-repo.** Suggest the CLAUDE.md addition;
  let the user do it.
- **Don't auto-run /sync-check.** The user does that next if they
  want.
- **Don't fabricate the git remote URL.** Verify with `git ls-remote`.
- **Don't ask the user for a local path.** Path is convention
  (`repos/<name>/`).
- **Don't proceed if `repos/<name>/` exists with a different remote.**
  That's a name collision — surface it to the user, let them rename
  or remove the conflicting checkout.

## When NOT to use this skill

- **Onboarding a new collaborator onto an existing instance** — use
  `bin/setup` instead. It clones every registered sub-repo
  automatically; no per-repo prompting.
- **Updating an existing sub-repo's notes** — edit `state/manifest.md`
  directly.
- **Removing a sub-repo** — edit the manifest, delete its state file,
  delete its entries from any active migrations. Optionally
  `rm -rf repos/<name>/` to remove the local clone. (No `/unregister`
  skill in v1.)

## What "done" looks like

`state/manifest.md` has a new entry for the sub-repo.
`state/sub-repos/<name>.md` exists with frontmatter populated.
`repos/<name>/` is a working git checkout (or the user has been told
why the clone failed and how to retry). The user has been shown the
suggested `CLAUDE.md` addition for that repo and can copy it in when
they choose. No commits to the orchestrator, no remote changes.
