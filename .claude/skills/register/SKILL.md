---
name: register
description: Register a new sub-repo with the orchestrator — adds it to state/manifest.md so it's included in /sync-check, migration notices, and status reports. Triggered by "/register", "register a sub-repo", "add the api repo", "new sub-kit", "track this repo".
---

# /register — Register a sub-repo with the orchestrator

Add a new sub-repo to `state/manifest.md`. After registration, the
sub-repo is included in `/sync-check`, eligible to receive migration
notices, and visible in `/status`.

## Behavior contract

- **Verify the path exists and is a git repo.** If not, refuse and
  explain.
- **Verify it's not already registered.** If a sub-repo with the same
  name or path is in the manifest, point that out.
- **Get the git remote from the repo, not from the user.** `git -C
  <path> remote get-url origin` — don't trust user-typed URLs that may
  drift from reality.
- **Don't update sub-repo `CLAUDE.md`** — recommend the addition to the
  user, but registering is a one-way orchestrator-side action. Sub-repo
  edits are the user's call to make.

## Process

1. Ask the user for:
   - Local path to the sub-repo
   - Role (api / ios / web / devops / other — one word)

2. Verify the path:
   - Path exists
   - Contains a `.git` directory or is a git worktree
   - Has a remote `origin`

3. Read remote URL, default branch
   (`git -C <path> symbolic-ref refs/remotes/origin/HEAD`).

4. Derive a name — usually the directory basename, but ask if it
   should be different.

5. Draft the new entry for `state/manifest.md`:

   ```markdown
   ## <name>
   - **Role:** <role>
   - **Local path:** <absolute path>
   - **Git remote:** <url>
   - **Default branch:** <main / master / etc>
   - **Registered:** <YYYY-MM-DD>
   - **Notes:**
   ```

6. Show to user. Confirm.

7. Append to `state/manifest.md` under the `## Sub-repos` heading
   (alphabetical or insertion order — be consistent with what's
   already there).

8. Suggest the user add this to the sub-repo's `CLAUDE.md`:

   ```markdown
   ## Macro architecture
   The cross-repo architectural truth lives in
   `<absolute-path-to-orchestrator>/`. Read `stack/inventory.md`,
   `contracts/`, `conventions/`, and any `features/<x>.md` matching
   current work before making cross-platform decisions.

   When working on a task, also check `.claude/active-migrations.md` in
   this repo — it lists open cross-repo migrations affecting this
   codebase.
   ```

   Don't write this for them. They commit it on their schedule.

## Style rules

- **One sub-repo per call.** Don't batch.
- **Confirm the path.** Show the absolute path you'll write before
  writing.
- **Honest about what registration does and doesn't do.** It adds a
  manifest entry. It doesn't connect anything magically.

## What you must NOT do

- **Don't write into the sub-repo.** Suggest the CLAUDE.md addition;
  let the user do it.
- **Don't auto-run /sync-check.** The user does that next if they
  want.
- **Don't fabricate the git remote URL.** Read it from the repo.

## When NOT to use this skill

- **Updating an existing sub-repo's notes** — edit `state/manifest.md`
  directly.
- **Removing a sub-repo** — edit the manifest and delete its entries
  from any active migrations. (No `/unregister` skill in v1.)

## What "done" looks like

`state/manifest.md` has a new entry for the sub-repo. The user has
been shown the suggested `CLAUDE.md` addition for that repo and can
copy it in when they choose. No commits, no remote changes.
