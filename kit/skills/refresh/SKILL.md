---
name: refresh
description: Fetch latest changes from origin for every registered sub-repo. Updates local git refs (no merge, no pull), reports ahead/behind + dirty flags, writes state/last-fetch.json so other skills know how fresh the data is. Triggered by "/refresh", "fetch sub-repos", "pull updates", "sync sub-repos", "what's new across our repos", "are we current".
---

# /refresh — Fetch sub-repo updates from origin

Bring local sub-repo refs current with their remote main branches.
Read-only against the remote. Writes only to local git refs and the
`state/last-fetch.json` timestamp.

This is the **session-start refresh**. Other orchestrator skills
(`/status`, `/sync-check`, `/roadmap`, `/tasks`) read sub-repo git
state under `repos/<name>/`. If those refs are stale, every downstream
skill produces a stale picture. `/refresh` keeps the picture honest.

## Behavior contract

- **Read-only against remotes.** `git fetch`, never `git pull` or
  `git merge`. The user controls when to update working trees.
- **Don't touch working trees.** No checkouts, no stashes, no
  rebases. Fetch is non-destructive; uncommitted changes are
  preserved.
- **Honest about failures.** Auth issues, network errors, missing
  remotes — surface per-repo with the actual error. Don't fake
  success.
- **Best-effort.** A single sub-repo's failure doesn't abort the
  whole refresh.
- **Skip remote-only mode.** If `repos/<name>/` doesn't exist on
  this machine, note it as `unfetched` and continue. Suggest
  `bin/setup` once at the end if any are missing.
- **Update the timestamp on completion.** Write
  `state/last-fetch.json` so downstream skills can reason about
  freshness.

## Process

1. Read `state/manifest.md`. Get name + `git_remote` + default
   branch for each registered sub-repo.

2. For each sub-repo:
   - If `repos/<name>/` doesn't exist → mark `unfetched`, continue.
   - Else:
     - `git -C repos/<name> fetch origin`
     - Current branch:
       `git -C repos/<name> rev-parse --abbrev-ref HEAD`
     - Ahead/behind vs `origin/<default-branch>`:
       `git -C repos/<name> rev-list --left-right --count HEAD...origin/<default-branch>`
     - Dirty flag:
       `git -C repos/<name> status --porcelain` — non-empty → dirty
     - Record per-repo: branch, ahead, behind, dirty, status (`ok`
       or error message)

3. Write `state/last-fetch.json`:

   ```json
   {
     "schema_version": 1,
     "fetched_at": "<ISO-8601>",
     "results": {
       "<name>": {
         "branch": "<current-branch>",
         "ahead": <n>,
         "behind": <n>,
         "dirty": true|false,
         "status": "ok" | "error" | "unfetched",
         "error": "<message or null>"
       }
     }
   }
   ```

4. Surface a summary to the user — one line per sub-repo, plus an
   error block if any failed.

## Output shape

```
═══ /refresh @ 2026-05-08T19:42 ═══

  api      [main +0 -3]      behind origin/main by 3 commits
  ios      [feat/x +2 -0]    ahead by 2 (uncommitted: yes)
  web      [main +0 -0]      current
  devops   unfetched         (run bin/setup to clone)

  errors:  none
```

When errors:

```
  api      [main]            ERROR: fatal: Authentication failed
```

## Style rules

- **One line per sub-repo.** Don't recite git output verbatim.
- **Surface dirty and behind.** These are the actionable signals —
  they tell the user where they need to act.
- **Don't suggest commands the user didn't ask for.** Mention
  `bin/setup` only if any sub-repo is `unfetched`; mention
  `git pull --rebase` only if a repo is `behind` AND `not dirty`.

## What you must NOT do

- **Don't merge or pull.** Fetch only. The user decides when to
  bring working trees forward.
- **Don't touch working trees.** No `git checkout`, no `git stash`,
  no `git rebase`.
- **Don't push or write to remotes.** This skill is read-only
  against remotes.
- **Don't trigger other skills.** `/refresh` is the building block;
  `/status` is what calls it (or warns about stale data). Stay
  focused.

## When NOT to use this skill

- **Want to compile state across repos** — use `/status` (it
  surfaces stale-fetch warnings and may call `/refresh` first if
  data is >24h old).
- **Want to update a working tree to the latest commit** — that's
  `git pull --rebase` inside the sub-repo, by hand. The orchestrator
  doesn't auto-pull.
- **Want to register a new sub-repo** — `/register`.
- **Want to clone all registered sub-repos for the first time** —
  `bin/setup`.

## What "done" looks like

`state/last-fetch.json` updated with the current run's results.
Per-repo summary surfaced with ahead/behind + dirty flags. Any
errors flagged clearly. No working-tree changes, no commits, no
remote writes.

## Freshness contract — used by other skills

`state/last-fetch.json`'s `fetched_at` timestamp is the source of
truth for "how stale is our sub-repo data." Other skills check it:

- **<24h:** data considered fresh; proceed silently.
- **24h–7d:** warn the user at the top of the output: "last refresh
  was <duration> ago — consider `/refresh`."
- **>7d:** strongly recommend `/refresh` before producing
  compilations; offer to run it.

Don't refuse to run when stale — just be clear about the staleness.
