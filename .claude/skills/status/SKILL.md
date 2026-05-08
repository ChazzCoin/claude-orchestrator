---
name: status
description: Render the macro picture at a glance — active migrations, sub-repo state, open questions, recently shipped, what looks stale or off. Daily-driver overview. Triggered by "/status", "where do things stand", "macro status", "what's happening", "give me the picture".
---

# /status — Macro picture at a glance

The 30-second answer to "where do things stand across the stack."

This skill **reads** state. It does not modify anything. It reflects
authored truth (`roadmap.md`, `open-questions.md`) and current sub-repo
reality (via `state/sync-status.md` if recent, or by querying directly).

Per CLAUDE.md: terse, honest. Surface what's real and what's stale; don't
narrate.

## Behavior contract

- **Read existing files; don't infer.** All sections below come from
  files that already exist or queries against registered sub-repos.
- **Don't run `/sync-check` automatically.** That regenerates state and
  may be expensive. Use the existing `state/sync-status.md` if it's
  recent (note its age); if it's stale, say so and suggest
  `/sync-check`.
- **Surface only what matters.** Don't recite all migrations and all
  open questions. Foreground active items, stale items, blocked items.
- **No commentary.** Render the picture; let the user decide what
  needs attention.

## What to read

In order:

1. `roadmap.md` — Now / Next / Recently shipped sections
2. `migrations/active/` — every file, parse frontmatter and per-repo
   state
3. `open-questions.md` — count active, surface ones tagged Blocking
4. `state/sync-status.md` — if it exists and is fresh (<24h)
5. `state/manifest.md` — to know which sub-repos are registered

## Output structure

```markdown
# 📍 Status — <YYYY-MM-DD>

## Now
*From `roadmap.md`.*
- <each Now item, one line>

## Active migrations (<count>)
- **<id>** [<status>] · <affects, with state symbols> <stale flag if applicable>
  - <one-line "what's changing">
- ...

*If stale migrations exist, surface them in a callout below the list.*

## Open questions (<count active>)
- <items tagged Blocking, with reason>
- *(N more deferred — see `open-questions.md`)*

## Recently shipped
*From `roadmap.md` "Recently shipped".*
- <last 3 entries, one line each>

## Sub-repo snapshot
*From `state/sync-status.md` if fresh; otherwise note staleness.*
- <repo>: <branch> @ <short-sha> · <N open PRs>
- ...

*If `state/sync-status.md` is older than 24h or absent: "Snapshot is
N days old — run `/sync-check` to refresh."*
```

If a section has no content, omit it — don't render a placeholder.

## Style rules

- **One screen.** If the output is longer than ~30 lines, you're
  summarizing too much. Cut.
- **No emoji decoration.** 📍 marks the report. ⚠ for stale or
  blocking. ⚪ 🟡 ✅ for migration state. Nothing else.
- **Dates are absolute.** "Opened 2026-05-07 (4 days ago)" — both forms.
- **No "let me know" closer.** End on the snapshot.

## What you must NOT do

- **Don't propose actions.** The user reads and decides.
- **Don't query gh / git** to refresh sub-repo state. That's
  `/sync-check`'s job. If state is stale, say so and stop.
- **Don't expand on items.** One line each. The user opens the file
  if they want depth.
- **Don't commit anything.** This skill is read-only.

## When NOT to use this skill

- **Want to refresh sub-repo state** → `/sync-check`.
- **Want to act on something** → `/migration`, `/decision`, `/feature`.
- **Want full context** → `/onboard`.
- **Looking at one specific migration** → just open the file.

## What "done" looks like

A single rendered status report. No file edits, no commits, no
follow-up questions. The user reads it and either acts or moves on.
