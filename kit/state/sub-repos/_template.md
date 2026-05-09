---
name: <sub-repo-name>
role: api | ios | web | devops | other
git_remote: <url>
default_branch: main
kit_enabled: true | false | unknown
kit_version: <foundation.json pinned_sha or "n/a">
registered: YYYY-MM-DD
last_synced_head: <sha or "never">
last_synced_at: YYYY-MM-DD or "never"
last_pulled_at: YYYY-MM-DD or "never"
kit_install_offered: <YYYY-MM-DD or "n/a">
kit_install_declined: <YYYY-MM-DD or "n/a">
---

# <name>

> Auto-managed by `/sync-check` and `/register`. Most fields above
> the **Notes** section are regenerated. **Notes** is the only
> section the user hand-edits — it's preserved across sync.

## Current state

- **Branch:** <local current branch>
- **HEAD:** <short-sha · YYYY-MM-DD · last commit subject>
- **Has uncommitted changes:** <yes / no>
- **Open PRs:** <count> ([gh link])
  - <#NNN — title · branch · updated YYYY-MM-DD>
- **Active task (from `<sub>/.claude/active.md`):** <task ID + title or "—" if file absent>
- **Sub-kit phase (from `<sub>/tasks/PHASES.md` if kit-enabled):** <phase name or "—">

## Capabilities

- **Read git log + PRs:** yes (any sub-repo)
- **Kit-enabled:** <true | false | unknown>
- **Read kit task system (`<sub>/tasks/`):** <yes if kit_enabled, else no>
- **Read advertisement (`<sub>/.claude/active.md`):** <yes if file exists, else no>
- **Has orchestrator back-pointer in `<sub>/CLAUDE.md`:** <yes | no | unknown>
- **Receives migration notices:** yes (any sub-repo)

## Affecting this repo

### Open migrations
*Populated from `migrations/active/` frontmatter `affects: [...]`.*
- _(populate)_

### Open features
*Populated from `features/` frontmatter `affects: [...]` if present.*
- _(populate)_

### Recent ADRs touching this repo
*Last 5 from `decisions/` mentioning this repo (manual or via skill).*
- _(populate)_

### Open risks tagged with this repo
*Populated from `risks/open/` frontmatter `affects: [...]`.*
- _(populate)_

### Open / unresolved incidents affecting this repo
*Populated from `incidents/` where status != postmortem-filed.*
- _(populate)_

## Orchestrator-authored work

PRs the orchestrator has opened in this repo (per
[`sub-projects.md`](../../.claude/sub-projects.md) "Git management"):

- _(populate by `/sync-check` via `gh pr list` filtered for branches
  matching `chore/orch-*`)_

Recently merged orchestrator PRs:

- _(populate)_

## Recent PR reviews on this repo

Orchestrator-grade reviews filed in `pr-reviews/` for PRs in this
repo (per [`pr-reviews/README.md`](../../pr-reviews/README.md)):

- _(populate by `/sync-check` from `pr-reviews/*.md` matching
  this `sub_repo`. Show last 5 by date with status emoji.)_

## Sub-kit advertisement *(if kit-enabled and `<sub>/.claude/active.md` present)*

Latest content read from `<sub>/.claude/active.md`:

```
_(populate — full file content or relevant excerpt)
```

## Shared context *(if `<sub>/.claude/shared/` present)*

Durable two-way per-repo files (see
[`templates/sub-repo-shared/README.md`](../../templates/sub-repo-shared/README.md)).
The orchestrator reads these on `/sync-check` and may append to
`shared/inbox.md`, `shared/notes.md`, or `shared/references.md`
when authoring features that affect this repo.

- **Architecture:** `<sub>/.claude/shared/architecture.md` — present: yes / no, last touched: <YYYY-MM-DD>
- **Repo inbox:** `<sub>/.claude/shared/inbox.md` — present: yes / no, last entry: <YYYY-MM-DD>, unread to orchestrator: <count>
- **Notes:** `<sub>/.claude/shared/notes.md` — present: yes / no, last touched: <YYYY-MM-DD>
- **References:** `<sub>/.claude/shared/references.md` — present: yes / no, last touched: <YYYY-MM-DD>

## Notes

*The orchestrator preserves this section across sync. Use for:
per-repo gotchas, special instructions, things to remember
specifically about this sub-repo. Empty is fine.*

_(manual)_
