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
- **Shared-channel scaffold is opt-in, via the standard PR flow.**
  After the manifest entry and clone succeed, offer to scaffold
  `<sub>/.claude/shared/` (the durable two-way per-repo channel —
  see [`kit/templates/sub-repo-shared/README.md`](../../templates/sub-repo-shared/README.md))
  via a `chore/orch-init-shared` PR. User declines or accepts; if
  declines, the channel can be created lazily later.
- **Preferences-aware.** Two known forks in this skill:
  - Step 11 (the shared-scaffold offer) → preference ID
    `auto-scaffold-shared-on-register` (low-risk).
  - The non-kit install-claude-kit offer (mentioned under
    [`sub-projects.md`](../../sub-projects.md) "Two flavors") →
    preference ID `auto-install-kit-on-non-kit-register` (low-risk).

  Both follow the recipe at
  [`.claude/preferences.md`](../../preferences.md) "Skill recipe
  at a known fork": read `state/preferences.md` first; apply
  silently with disclosure if found; otherwise ask, log to
  `state/decision-log.md`, and run the streak-threshold offer.

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

    Before working a task, follow the orchestrator's developer
    pre-flight checklist at `../../.claude/governance/dev-preflight.md`
    (relative to this sub-repo's root; the orchestrator is two
    directories up).

    On every session start, read:
    - `.claude/active-*.md` — orchestrator notices (open migrations,
      open features, and any other auto-managed concerns affecting
      this repo)
    - `.claude/shared/inbox.md` — durable repo inbox (messages from
      the orchestrator or prior sessions)
    - `.claude/shared/notes.md` — durable shared notes for this repo
    ```

11. **Offer to scaffold the durable shared-context channel.** This
    is a preferences-aware fork (ID
    `auto-scaffold-shared-on-register`, low-risk).

    **Implementation:** invoke `bin/preferences-check` for the
    mechanical steps. The script reads `state/preferences.md`,
    writes `state/decision-log.md`, and returns JSON the skill
    consumes. The skill handles the user-facing prompts +
    confirmations.

    **Preferences-recipe step 1 (read):**
    ```sh
    bin/preferences-check resolve auto-scaffold-shared-on-register
    ```
    Returns JSON: `{found: bool, ..., disclosure_message: "..."}`.
    If `found: true`:
    - Apply silently — proceed to scaffold without asking.
    - **First apply per session:** print the JSON's
      `disclosure_message`.

    If not found, ask:

    > "Want me to scaffold `<sub>/.claude/shared/` now? That's the
    > durable two-way per-repo channel — `architecture.md`,
    > `inbox.md`, `notes.md`, `references.md`. I'll open a PR
    > (`chore/orch-init-shared`) so you can review before merge.
    > You can also skip — the files can be created lazily later."

    **Preferences-recipe step 4 (log):** after the user answers,
    invoke:
    ```sh
    bin/preferences-check log auto-scaffold-shared-on-register \
      <yes|no> --context "sub-repo: <name>"
    ```
    Returns JSON: `{logged: true, streak: N, should_offer: bool,
    offer_prompt: "..."}`.

    **Preferences-recipe step 5 (offer threshold):** if
    `should_offer: true`, print the JSON's `offer_prompt` to the
    user. On `yes`:
    ```sh
    bin/preferences-check capture auto-scaffold-shared-on-register \
      <yes|no> --evidence "<user's most recent answer>" \
      --source offer-accepted
    ```
    On `no`/silence: nothing further; the script's `log` call
    already updated the cooldown.

    If user declines the original scaffold (regardless of capture
    flow), skip to step 12.

    If user accepts (or preference applied), render each of the
    four files from
    [`kit/templates/sub-repo-shared/`](../../templates/sub-repo-shared/)
    by substituting placeholders:

    - `{{REPO_NAME}}` → the sub-repo's name
    - `{{TIMESTAMP}}` → current ISO-8601 UTC
    - `{{DATE}}` → current `YYYY-MM-DD`

    Then, in `repos/<name>/`:

    ```sh
    git -C repos/<name> checkout main           # or default branch
    git -C repos/<name> pull
    git -C repos/<name> checkout -b chore/orch-init-shared
    mkdir -p repos/<name>/.claude/shared
    # write the four rendered files into .claude/shared/
    git -C repos/<name> add .claude/shared/
    git -C repos/<name> commit -m "orch: scaffold .claude/shared/ for durable two-way context"
    git -C repos/<name> push -u origin chore/orch-init-shared
    gh pr create -R <git_remote> --title "orch: scaffold .claude/shared/" \
      --body "Bootstraps the durable two-way per-repo context channel.

    Files added:
    - .claude/shared/architecture.md — living architecture doc
    - .claude/shared/inbox.md — durable repo inbox
    - .claude/shared/notes.md — durable shared notes
    - .claude/shared/references.md — curated URLs

    See orchestrator governance: kit/templates/sub-repo-shared/README.md
    and kit/governance/dev-preflight.md."
    ```

    Show the user the PR URL. They merge when ready. **Don't
    auto-merge.**

12. Done. Summarize what was created (manifest entry, state file,
    clone, optional shared scaffold PR) and surface next steps.

## Style rules

- **One sub-repo per call.** Don't batch.
- **Confirm before writing.** Show the manifest entry and the clone
  destination before executing.
- **Honest about what registration does and doesn't do.** It adds a
  manifest entry, creates a state file, and clones the repo. It does
  not modify anything inside the cloned repo.

## What you must NOT do

- **Don't write into the sub-repo *except* via the optional shared
  scaffold PR (step 11), and only with explicit user consent.** All
  other writes — including the suggested CLAUDE.md addition — are
  the user's to make.
- **Don't auto-merge the shared scaffold PR.** Open it, surface the
  URL, let the user review and merge.
- **Don't auto-run /sync-check.** The user does that next if they
  want.
- **Don't fabricate the git remote URL.** Verify with `git ls-remote`.
- **Don't ask the user for a local path.** Path is convention
  (`repos/<name>/`).
- **Don't proceed if `repos/<name>/` exists with a different remote.**
  That's a name collision — surface it to the user, let them rename
  or remove the conflicting checkout.
- **Don't scaffold `shared/` if it already exists in the sub-repo's
  default branch.** Check before the offer; if any of the four
  files already exist, skip the offer and tell the user.

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
they choose. The shared-channel scaffold has been offered; if
accepted, a `chore/orch-init-shared` PR is open in the sub-repo
awaiting user review. No commits to the orchestrator, no automatic
merges in the sub-repo.
