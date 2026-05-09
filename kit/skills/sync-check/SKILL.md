---
name: sync-check
description: Pull current state across registered sub-repos and regenerate state/sync-status.md. Surfaces drift between sub-repo reality and active migrations. Triggered by "/sync-check", "refresh status", "what's the actual state", "are we in sync", "drift check".
---

# /sync-check — Pull and reflect sub-repo state

Regenerate `state/sync-status.md` from authoritative sources (`gh`,
`git log`, sub-repo `.claude/active.md` files if they exist). Surface
drift between sub-repos and the active migrations.

This skill **regenerates** state. It does not author anything new.

**Output pattern:** [Pattern 17 — Git branch overview](../../output-catalogue.md#17--git-branch-overview)
for the per-sub-repo block +
[Pattern 6 — Severity audit](../../output-catalogue.md#6--severity-audit)
shape for the Drift section (untracked branches and untracked commits
ordered by impact). Drift is the headline — surface it loudly when it
exists.

## Behavior contract

- **Read `state/manifest.md` first.** Only query registered sub-repos.
  Unregistered repos in adjacent directories are not the orchestrator's
  business.
- **Each query is fast and idempotent.** No mutation.
- **Surface drift loudly.** A sub-repo with commits beyond what any
  migration tracks is a signal — note it explicitly.
- **Note query failures honestly.** If `gh` isn't authenticated or a
  repo can't be reached, say so; don't write fake state.
- **Surface stale-fetch warning.** Read `state/last-fetch.json` at
  the start. If missing or >24h old, prepend a warning to the
  output: "Last refresh: <duration> ago — consider `/refresh` for
  fresh sub-repo state." Don't refuse to run.
- **Handle remote-only mode gracefully.** Sub-repos live at
  `repos/<name>/` by convention. If that directory doesn't exist on
  this machine (e.g. mobile, or before `bin/setup` ran), fall back
  to GitHub API queries against the manifest's `git_remote` for
  branch / HEAD / last-commit. Don't error — surface remote-only
  mode in the output and continue.

## Process

1. Read `state/manifest.md`. For each sub-repo, get name, role,
   `git_remote`, default branch. The working tree (if present on
   this machine) lives at `repos/<name>/` — convention, not
   configuration.

2. For each sub-repo, gather:
   - **If `repos/<name>/` exists locally:**
     - branch: `git -C repos/<name> rev-parse --abbrev-ref HEAD`
     - HEAD short SHA + last commit date:
       `git -C repos/<name> log -1 --format='%h %ad %s' --date=short`
     - Sub-kit signal: read `repos/<name>/.claude/active.md` if it
       exists (this is what a sub-kit may write to advertise its
       current task).
   - **Otherwise (remote-only mode):**
     - Use the manifest's default branch as the surveyed branch.
     - HEAD: `gh api repos/<owner>/<repo>/branches/<default> --jq .commit.sha`
     - Last commit date: same response, `.commit.commit.author.date`
     - Sub-kit signal: try
       `gh api repos/<owner>/<repo>/contents/.claude/active.md`;
       if 404, no advertisement.
   - **Always:** open PRs
     (`gh pr list -R <owner>/<repo> --json number,title,headRefName,updatedAt`).

3. Read `migrations/active/`. For each migration, gather affected repos
   and per-repo state.

4. Compute drift:
   - Sub-repo branches that aren't main and aren't tracked by any open
     migration's per-repo plan
   - Sub-repos with new commits since last sync that aren't covered by
     a migration
   - Migrations marked `in-progress` for ≥14 days with one or more
     repos at ⚪ or 🟡

5. Write `state/sync-status.md` with:
   - Generation timestamp
   - Per sub-repo: branch, HEAD SHA + date, open PR count, current
     active task if known
   - Per active migration: per-repo state, staleness flag
   - Drift section: untracked branches, untracked commits, stale
     migrations

6. Surface a short summary to the user — counts of each issue type,
   what to look at first.

## Output structure for state/sync-status.md

```markdown
# Sync status

**Generated:** <YYYY-MM-DD HH:MM>

## Sub-repos

### api
- **Branch:** main
- **HEAD:** <short-sha> · <YYYY-MM-DD> · <last commit subject>
- **Open PRs:** <N>
  - #234 — <title> · <branch> · updated <date>
- **Active task (sub-kit signal):** <from .claude/active.md or "—">

### ios
...

## Active migrations

### 2026-05-07-user-add-tenant-id
- **Status:** in-progress
- **Per-repo:** api ✅ · ios 🟡 PR #89 · web ⚪ · devops ✅
- **Age:** 4 days

...

## Drift

*Items that don't fit cleanly under any migration.*

- **api:main** has 3 commits since last sync not associated with any
  open migration
- ...

*If no drift, write "None — registered sub-repos and active migrations
are aligned."*
```

## Style rules

- **Timestamps absolute.** "Generated 2026-05-07 14:32" not "just now".
- **One row per fact.** Don't merge "branch + sha + date" into one
  line if it makes things less readable.
- **Drift is the headline.** If drift exists, it's the most important
  output of this run.

## What you must NOT do

- **Don't make remote changes.** No PR comments, no branch pushes, no
  commits. Read-only against external systems.
- **Don't write into sub-repos.** This skill writes to
  `state/sync-status.md` only.
- **Don't infer drift from staleness alone.** "No commits in 30 days"
  isn't drift; "commits exist but no migration tracks them" is.

## When NOT to use this skill

- **Want a quick read** without refreshing sub-repo data → `/status`.
- **Want to register a new sub-repo** → `/register`.
- **Want to update a migration** based on what you find → `/migration`
  in update mode.

## What "done" looks like

`state/sync-status.md` regenerated with current data. A short summary
shown to the user surfacing drift count, stale migrations, and any
query failures encountered. No other writes.
