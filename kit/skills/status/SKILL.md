---
name: status
description: Compile the macro picture across the orchestrator and every sub-repo. Renders inbox count + headlines, roadmap Now, upcoming events, active migrations, open questions, per-sub-repo state (branch + ahead/behind + sub-kit advertisement), drift, recently shipped. Pure read; the daily-driver overview. Triggered by "/status", "where do things stand", "macro status", "what's happening", "give me the picture".
---

# /status — Macro picture across the stack

The compiled view. Pulls orchestrator state, sub-repo git + claude-kit
state, the user's inbox, upcoming events, active migrations, and
recent shipped work into one rendered report.

This skill is **read-only**. It writes nothing — just compiles from
artifacts already on disk.

**Output pattern:** composition of
[Pattern 28 — Stats card grid](../../output-catalogue.md#28--stats-card-grid)
for headline counts +
[Pattern 17 — Git branch overview](../../output-catalogue.md#17--git-branch-overview)
for the sub-repos block (per-repo state line with glyph + branch +
ahead/behind + sub-kit task) +
[Pattern 23 — Activity timeline](../../output-catalogue.md#23--activity-timeline)
elements for migrations / events / recently shipped. Sections marked
by `▸` glyph; emoji used sparingly (`⚪ 🟡 ✅` for migration state,
`⚠` for stale / blocking).

## Behavior contract

- **Identity required.** Read `.claude/local-config.json` for `me`
  and `inbox_last_read`. If missing, abort with: "no
  local-config.json — run `bin/setup` to set your identity."
- **Surface fetch staleness.** Read `state/last-fetch.json`. If
  missing or older than 24h, prepend a warning. If older than 7d,
  recommend `/refresh` before reading further.
- **Read existing files; don't infer.** All sections come from
  files or trivial reads (`repos/<name>/.claude/active.md`).
- **Don't auto-refresh.** `/refresh` and `/sync-check` are
  separate. If their data is stale, say so. Don't trigger them
  silently.
- **Surface only what matters.** Foreground active, blocking,
  stale. Don't recite the entire roadmap.

## What to read

| Source | Powers |
|---|---|
| `.claude/local-config.json` | identity (`me`, `inbox_last_read`) |
| `state/last-fetch.json` | freshness gate + sub-repo branch / ahead / behind / dirty |
| `state/sync-status.md` | sub-repo PR counts (if /sync-check recent) |
| `state/inbox/<me>.md` | unread inbox count + headlines |
| `roadmap.md` | Now / Recently shipped |
| `events.md` | Upcoming (next 14 days) |
| `migrations/active/*.md` | active migrations + per-repo state symbols |
| `features/*.md` (status: planning or in-progress) | open feature count |
| `open-questions.md` | Blocking-tagged items |
| `state/manifest.md` | registered sub-repos |
| `repos/<name>/.claude/active.md` | sub-kit advertisement (if kit-enabled) |
| `state/preferences.md` | high-risk preferences (always surfaced) |
| `state/decision-log.md` | forks approaching auto-offer threshold (streak ≥ 2 same-answer) |
| `proposals/<name>/backlog/` | per-repo proposal counts (drafts not yet promoted) |

No `gh` calls, no `git fetch`. Pure read against on-disk state.

## Output shape

```
═══ Status @ <YYYY-MM-DD HH:MM> · <me>@<machine_id> ═══

  [optional one-line freshness warning]

▸ Inbox (<N unread>)
    <from> · <subject>
    <from> · <subject>

▸ Now
    <each roadmap "## Now" bullet>

▸ Upcoming events (<N in 14d>)
    <YYYY-MM-DD> · <title> (in <N> days)

▸ Active migrations (<count>)
    <id> [<status>] · <affects-with-symbols> [⚠ if stale]
      <one-line summary>

▸ Open questions — Blocking
    <Blocking items, one line each>
    (<N> more open — see open-questions.md)

▸ Sub-repos
    <name>  <branch> +<ahead> -<behind>  [dirty]  <N PRs>  <sub-kit task or —>
    <name>  remote-only                                     (no local clone)

▸ Proposals (<N drafts across M repos>)
    <repo>  <count> drafts  ·  <count> phases proposed
    (only repos with at least one proposal in backlog/)

▸ High-risk preferences (<N>)
    <id>  <decision>  · set <YYYY-MM-DD> <handle>
    (always surfaced — high-risk capture is sticky)

▸ Approaching auto-offer (<N>)
    <id>  yes <K> / no <M> (streak: <K> yes)  · 1 more for offer
    (low-risk forks with streak == 2; advisory)

▸ Drift
    <commits in sub-repos not associated with any active migration>

▸ Recently shipped
    <last 3 from roadmap "## Recently shipped">
```

If a section has no content, omit it entirely. Don't render
placeholders. Inbox section omits when 0 unread.

## Per-section logic

### Freshness warning

`state/last-fetch.json`:
- absent → "no fetch on this machine yet — run `/refresh`."
- `fetched_at` < 24h ago → no warning.
- 24h–7d → "last refresh: `<duration>` ago — `/refresh` for fresh data."
- > 7d → "last refresh: `<duration>` ago — strongly recommend
  `/refresh` before acting on this report."

### Inbox

1. Read `state/inbox/<me>.md` if present.
2. Parse `## <ts> — <from> — <subject>` headings.
3. Filter to `ts > inbox_last_read` (or all if `inbox_last_read`
   is null).
4. Show count + one-liner per unread (max 5; "and N more"
   otherwise).
5. **Don't mark as read.** That's `/inbox`'s job. `/status` is a
   notice.

### Now

1. Open `roadmap.md`.
2. Locate `## Now` heading; render its bullets verbatim.

### Upcoming events

1. Open `events.md`.
2. Locate `## Upcoming` heading.
3. Parse entries (`## <YYYY-MM-DD> · <title>`).
4. Filter: date ≤ today + 14 days; date ≥ today.
5. Sort soonest-first.
6. Render `<date> · <title> (in <N> days)`.

### Active migrations

1. List `migrations/active/*.md`.
2. Parse frontmatter for `id`, `status`, `affects`, `opened`.
3. Read per-repo state line for ⚪ 🟡 ✅ symbols.
4. Stale flag (⚠): `opened ≥ 14 days ago` AND any per-repo state
   is ⚪ or 🟡.

### Open questions — Blocking

1. Open `open-questions.md`.
2. Find entries tagged `Blocking` (or under a `## Blocking`
   subheading).
3. Render one line each.
4. Note count of remaining open (non-blocking) questions.

### Proposals

1. Walk `proposals/<name>/` directories.
2. For each repo with `proposals/<name>/backlog/*.md`, count drafts.
3. For each `proposals/<name>/PHASES.md`, count `## Phase ` headings
   (skip those marked `*(retired …)*` or `*(promoted …)*`).
4. Render only repos with non-zero counts. If all repos have zero,
   omit the section.

### High-risk preferences

1. Read `state/preferences.md`. Parse `## Active preferences` for
   entries with `**Tier:** high-risk`.
2. Render every high-risk preference, one per line. **Always
   surface** even when the list is short — the discipline doc says
   high-risk capture is "sticky," continuously visible.
3. If zero high-risk preferences are captured, omit the section.

### Approaching auto-offer

1. Read `state/decision-log.md`. Parse `## Active` per-fork sections.
2. For each fork:
   - Skip if tier is high-risk (no auto-offer regardless of streak).
   - Skip if active cooldown is set (`Last offer:` field shows
     non-expired cooldown).
   - Compute streak from header `**Tally:** ... (streak: <K> <value>)`.
   - Include the fork only if streak == 2 (one short of auto-offer
     threshold of 3).
3. Render advisory: "1 more for offer." Helps the user know the
   orchestrator is about to start asking-to-remember.

### Sub-repos

1. Read `state/manifest.md` for the registered list.
2. For each entry:
   - Try `state/last-fetch.json.results[<name>]` for branch +
     ahead + behind + dirty.
   - If status is `unfetched` → render `<name>  remote-only`.
   - Read `repos/<name>/.claude/active.md` if it exists; extract
     the active task line.
   - Read `state/sync-status.md` for cached PR count (if recent;
     otherwise show `?` for PRs).
3. Compact one-liner per sub-repo. Align columns for skim-readability.

### Drift

1. For each sub-repo's current branch (from last-fetch.json):
   - If branch == default → skip (no drift signal).
   - Else: check active migrations' `affects` for this
     `<repo>/<branch>` pair. If no migration covers it, it's drift.
2. For each sub-repo `ahead > 0` on default branch with no
   active migration referencing it → drift.
3. List drift items. If 0 → omit section.

### Recently shipped

1. Open `roadmap.md`; locate `## Recently shipped`.
2. Render last 3 entries.

## Style rules

- **One screen.** If output exceeds ~40 lines, cut. This runs
  often.
- **Sparse emoji.** `⚪ 🟡 ✅` for migration state, `⚠` for stale
  / blocking, `▸` for section headers. No other decoration.
- **Absolute dates + relative deltas.** "Opened 2026-05-07 (4 days
  ago)." Both forms.
- **Aligned columns** in the Sub-repos section — easier to scan.
- **No closer.** End on the last section.

## What you must NOT do

- **Don't auto-run `/refresh` or `/sync-check`.** Recommend them if
  data is stale; the user invokes.
- **Don't propose actions.** The user reads and decides.
- **Don't expand items.** One line each.
- **Don't write anything.** No file edits, no `inbox_last_read`
  updates, no commits.
- **Don't make remote calls.** No `gh`, no `git fetch`. Pure
  on-disk read.

## When NOT to use this skill

- **Want fresh sub-repo data** → `/refresh`.
- **Want deeper sub-repo state (PRs, drift detection)** →
  `/sync-check`.
- **Want compiled cross-repo roadmap view** → `/roadmap`.
- **Want compiled cross-repo task view** → `/tasks`.
- **Want to actually read and ack inbox** → `/inbox` (this just
  surfaces unread count + headlines).
- **Want full onboarding context** → `/onboard`.

## What "done" looks like

A single compiled status report rendered. No file edits, no remote
calls. ~30-40 lines. The user reads it and either acts or moves on.
