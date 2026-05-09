---
name: refresh
description: Fetch latest changes from origin for every registered sub-repo. Runs bin/refresh — a deterministic shell script that handles the entire flow. Updates state/last-fetch.json so other skills know how stale the data is. Triggered by "/refresh", "fetch sub-repos", "pull updates", "sync sub-repos", "are we current", "what's new across our repos".
---

# /refresh — Fetch sub-repo updates from origin

Runs `bin/refresh`. The script handles the entire flow
deterministically; this skill is a thin wrapper that invokes it and
surfaces output.

The flow is locked: read manifest → for each cloned sub-repo
`git fetch origin` → record ahead / behind / dirty → write
`state/last-fetch.json` → print summary. Read-only against remotes.
Never pulls, never merges, never touches working trees.

## Process

1. Run `bash <orchestrator-root>/bin/refresh`.
2. Show the script's stdout to the user verbatim.
3. If exit code is non-zero, surface the error context.
4. **Don't reinterpret the script's behavior.** The script is the
   spec.

## What the user can ask after

- *"What's in `state/last-fetch.json`?"* — read and show the JSON.
- *"Why did `<repo>` fail?"* — read that result entry's `error`
  field.
- *"Should I run `/sync-check`?"* — only suggest if any repo is
  behind on its default branch or dirty.

## What you must NOT do

- **Don't reimplement the flow in Claude-side logic.** If
  `bin/refresh` is broken or misbehaving, fix the script and
  propagate via `/sync`. Don't paper over with Claude
  interpretation — that defeats the lockdown.
- **Don't run `git fetch` directly bypassing the script.** The
  script is the single source of truth for what `/refresh` does.
- **Don't auto-pull or auto-merge.** The script doesn't; this skill
  doesn't either.

## When NOT to use this skill

- **Want compiled state across repos** → `/status`.
- **Want deeper sub-repo state (PRs, drift detection)** →
  `/sync-check`.
- **Want to update working tree to latest** → `git pull --rebase`
  inside the sub-repo, by hand.
- **Want to clone sub-repos for the first time** → `bin/setup`.

## What "done" looks like

`state/last-fetch.json` updated with the current run's results.
Summary surfaced to the user. No working-tree changes, no commits,
no remote writes.

## Freshness contract — used by other skills

`state/last-fetch.json`'s `fetched_at` timestamp is the source of
truth for "how stale is our sub-repo data." Other skills check it:

- **<24h:** fresh; proceed silently.
- **24h–7d:** warn at top of output: "last refresh was `<duration>`
  ago — consider `/refresh`."
- **>7d:** strongly recommend `/refresh` before producing
  compilations.

Don't refuse to run when stale — just be clear about the staleness.
