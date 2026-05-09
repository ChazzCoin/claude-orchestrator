---
name: preferences
description: Inspect, set, or revoke operating preferences — recurring decisions the orchestrator has learned and no longer asks about — and inspect the decision log (running tally of answers at known forks that informs auto-offer-to-capture). Triggered by "/preferences", "/prefs", "what have you remembered", "show my preferences", "show my tallies", "what's my tally on X", "stop asking me about X", "always do X", "from now on X", "remember to X", "forget the preference about X", "revoke preference X", "clear the log for X".
---

# /preferences — Operating preferences

The user-facing skill for the preferences system documented in
[`.claude/preferences.md`](../../preferences.md). Use this to
inspect what the orchestrator has learned, set a new preference,
or revoke one.

The point: less friction. Every preference set here is one fewer
question the orchestrator asks on every subsequent run.

**Output pattern:** [Pattern 4 — Sprint task board](../../output-catalogue.md#4--sprint-task-board)
adapted for `list` mode (compact rows: id, decision, tier, set
date) + [Pattern 1 — Hero completion card](../../output-catalogue.md#1--hero-completion-card)
on `set` and `revoke` confirmation.

---

## Modes

Detect from user input:

- **List** — "/preferences", "what have you remembered", "show my
  preferences" → enumerate active preferences AND the decision log
  (forks that have been asked but not yet captured, with their
  current tally).
- **Show** — "/preferences show `<id>`", "tell me about preference X",
  "what's my tally on X" → show one preference's full entry with
  evidence, OR if no preference exists, the decision log's history
  for that fork.
- **Set** — "always do X", "from now on X", "stop asking about X",
  "remember this", "/preferences set `<id>` `<value>`" → record
  a preference. Path 1 of the capture protocol (explicit signal).
- **Revoke** — "stop auto-X", "forget the preference about X",
  "/preferences revoke `<id>`" → revoke a preference. The fork
  resumes accumulating in the decision log.
- **Clear-log** — "/preferences clear-log `<id>`", "clear the log
  for X", "reset the tally on X" → wipe the active history for one
  fork in the decision log (cooldowns reset, streak resets to 0).
  Useful when the tally has gone stale or a recent decision was an
  outlier.

If ambiguous, ask which.

The auto-offer-to-capture flow (Path 2 of the capture protocol) is
**not** a mode of this skill — it's implemented by the asking
skills (`/register`, `/feature`, `/migration`, etc.) right after
they get an answer to a known-fork question. See the discipline
doc at [`.claude/preferences.md`](../../preferences.md) "Decision
log discipline" for the recipe each asking skill follows.

---

## Files

Two files this skill reads, both per-instance state:

- **`state/preferences.md`** — captured preferences. This skill is
  the only thing that writes to it. Format spec in
  [`.claude/preferences.md`](../../preferences.md) "What a
  preference is" + template at
  `bootstrap/preferences.md.template`.
- **`state/decision-log.md`** — running tally of decisions at
  known forks. This skill reads + prunes (when a preference is
  captured, the fork's section moves from Active to Captured
  archive). Asking skills append entries directly per the format
  in `bootstrap/decision-log.md.template` and the discipline at
  [`.claude/preferences.md`](../../preferences.md) "Decision log
  discipline."

Both files are committed (per-instance state, not gitignored).
Hand edits are tolerated but discouraged — the format is
machine-readable and skills overwrite on subsequent writes.

---

## Mode: List

### Process

1. Read `state/preferences.md` and `state/decision-log.md`. Either
   may be absent (fresh instance); both absent → "no preferences
   captured and no decisions logged yet."
2. Parse:
   - Preferences: `## Active preferences` and (if asked)
     `## Revoked preferences`.
   - Decision log: `## Active` (forks not yet captured) and (if
     asked) `## Captured (archive)`.
3. Render in two sections — captured first, tracked-but-not-captured
   second. The second is the "almost there" view; the user can see
   what's accumulating toward auto-offer.

   ```
   captured preferences (N):

     auto-scaffold-shared-on-register   yes  · low-risk   · 2026-05-08 chazz
     auto-install-kit-on-non-kit-register yes · low-risk   · 2026-05-08 chazz
     auto-merge-orch-prs                no   · high-risk  · 2026-05-09 chazz

   tracked decisions (M, not yet captured):

     auto-write-active-features-notices    yes 4 / no 0 (streak: 4 yes) — 1 more for auto-offer
     auto-push-orch-branches               yes 2 / no 1 (streak: 2 yes) — high-risk, no auto-offer
     auto-write-active-migrations-notices  yes 1 / no 0 (streak: 1 yes) — needs 2 more

   high-risk preferences are also surfaced in /status.
   revoke any with: /preferences revoke <id>
   clear a tally with: /preferences clear-log <id>
   ```

4. If user asked for revoked / archive too ("/preferences list
   all"), append the revoked-preferences and captured-archive
   sections.

### Style

- One line per item.
- Tier shown explicitly so high-risk is visible at a glance.
- Decision-log entries show the gap to next auto-offer when
  applicable ("1 more for auto-offer"); for high-risk forks,
  state "no auto-offer" so the user knows why the streak isn't
  triggering anything.
- Don't show evidence or full history in list mode — that's `show`.

---

## Mode: Show

### Process

1. Identify the `<id>`. If user said "tell me about auto-scaffolding,"
   match against the known IDs from
   [`.claude/preferences.md`](../../preferences.md) "Known
   preferences" table; if ambiguous, list candidates.
2. Look in **`state/preferences.md`** first.
   - If found → render the full preference entry: id, decision,
     scope, tier, set, evidence verbatim, revoke command. If the
     preference is **stale** (set >180 days ago), flag and suggest
     re-confirm or revoke.
   - If found, also check `state/decision-log.md` "Captured
     (archive)" for the fork's pre-capture history and surface a
     summary line ("captured after streak of 5 yes; full history
     in archive").
3. If not in preferences, look in **`state/decision-log.md`** "Active":
   - If found → render the fork's tally header + the full History
     list. Surface the auto-offer status: "next offer at streak 3
     (currently 2)" or "in cooldown: 3 decisions remaining" or
     "high-risk — no auto-offer regardless of streak."
   - If not found → "no preference captured and no decisions
     logged for `<id>`. The orchestrator will ask normally next
     time it hits this fork."

---

## Mode: Set

### Behavior contract

- **Verify the ID is known.** Match the user's intent against the
  registry in
  [`.claude/preferences.md`](../../preferences.md) "Known
  preferences." If the user is trying to set a preference for an
  unknown ID, surface that and ask: is this a new known
  preference (which would require updating the registry first), or
  did they mean a different ID?
- **Verify the tier permits capture.** `never-suppressible` IDs are
  refused with explanation.
- **For high-risk tier, confirm explicitly.** Re-state the
  preference, the consequences of auto-application, and require an
  unambiguous yes before writing.
- **Capture evidence.** Verbatim quote of the user's statement that
  established the preference.
- **Don't auto-commit.** Write the file; user reviews and commits.

### Process

1. **Identify the preference ID** from user input. If they used
   natural language ("always scaffold the shared/ thing"), map to
   the closest known ID and confirm.
2. **Identify the value.** Usually `yes` / `no`; can be a short
   string for non-binary preferences.
3. **Look up the tier** from the known-preferences registry.
4. **If never-suppressible:** refuse, explain, stop.
5. **If high-risk:** confirm explicitly:

   > "You're setting `<id>` to `<value>`. This is a **high-risk**
   > preference — going forward, the orchestrator will [describe
   > what auto-application means], without asking. It will be
   > surfaced in `/status` so it stays visible. Confirm? [yes / no]"

   Wait for unambiguous yes.

6. **For low-risk:** brief confirm:

   > "Save `<id>: <value>`? Revoke later with `/preferences revoke
   > <id>`. [yes / no]"

7. **Capture evidence.** Use the user's most recent statement that
   established this preference, verbatim.
8. **Read** `state/preferences.md`.
9. **Write or update the entry** in the `## Active preferences`
   section, sorted alphabetically by ID:

   ```markdown
   ### <id>

   - **Decision:** <value>
   - **Scope:** global  *(or whatever applies)*
   - **Tier:** <tier>
   - **Set:** <YYYY-MM-DD> by <handle from local-config.json `me`>
   - **Evidence:** "<verbatim quote>"
   - **Revoke:** `/preferences revoke <id>`
   ```

10. **If updating an existing entry**, the prior `Set:` and
    `Evidence:` are preserved by appending a new line to a `History:`
    field rather than overwriting:

    ```
    - **History:**
      - <prev-date> <handle>: "<prev evidence>" — <prev decision>
    ```

11. **Tell the user** what was written. Show the entry. Surface
    the revoke command.

---

## Mode: Revoke

### Behavior contract

- **Confirm before deleting.** Even though revocation is reversible
  (set again), the audit trail moves to the Revoked section.
- **Always log to revoked section, never just delete.** Auditability
  is the point.

### Process

1. Identify the `<id>`. Same matching as Show.
2. Read the entry. Show it.
3. Confirm:

   > "Revoke `<id>`? It moves to the Revoked section; the
   > orchestrator will start asking again. The decision log will
   > resume accumulating from the next answer. [yes / no]"

4. Optionally ask for a one-line reason (helpful future audit).
5. On confirm, in `state/preferences.md`:
   - Remove the entry from `## Active preferences`.
   - Append to `## Revoked preferences`:

     ```markdown
     ### <id> *(revoked)*

     - **Was:** <decision> (set <YYYY-MM-DD> by <handle>)
     - **Revoked:** <YYYY-MM-DD> by <handle>
     - **Reason:** <one line or omitted>
     ```

6. In `state/decision-log.md`:
   - Move the fork's entry from `## Captured (archive)` back to
     `## Active`, but with a fresh empty `### History` list. The
     archive entry is preserved.
   - Aggregate header reset: tally 0/0, streak 0, last asked
     "never since revoke," last offer "never since revoke."

7. Tell the user. Show the revoked preference entry and confirm
   the log is reset.

---

## Mode: Clear-log

### Behavior contract

- **Affects only the decision log, not preferences.** Use when a
  fork's tally has gone stale or a recent batch of decisions
  shouldn't influence future offers.
- **Confirm before clearing.** Reversible only by re-accumulating
  history; the cleared entries are not preserved.

### Process

1. Identify the `<id>`. If absent in the log, tell the user
   nothing's tracked for it and stop.
2. Read the fork's section in `state/decision-log.md` "Active." Show
   the current tally + recent History.
3. Confirm:

   > "Clear the decision log for `<id>`? Tally resets to 0/0,
   > streak resets, cooldowns reset. Preferences are not affected.
   > [yes / no]"

4. On confirm:
   - Remove the History entries.
   - Reset the aggregate header.
   - Optionally append a one-line audit comment to the section
     body: `*Cleared <YYYY-MM-DD> by <handle>: <reason>*`.

5. Tell the user. Show the cleared section.

---

## Style rules

- **Don't auto-commit.** All writes leave the file dirty for the
  user.
- **Verbatim evidence.** Capture the user statement as it was said,
  not paraphrased. Quote it.
- **Short confirmations.** The whole point is friction reduction —
  the confirmation prompt itself is short.
- **One question per turn.** If you need to disambiguate the ID,
  ask. If you need to confirm a high-risk capture, that's a
  separate turn. Don't bundle.
- **No emoji.** ID + decision + tier + date is the visual.

---

## What you must NOT do

- **Don't capture preferences for `never-suppressible` IDs.** Even
  if the user insists. Explain why and stop.
- **Don't auto-capture from the decision log without an offer.**
  The auto-offer flow (Path 2) requires an explicit user yes per
  offer. The threshold is the trigger to *ask*, not to silently
  capture.
- **Don't auto-offer for high-risk forks.** Regardless of streak
  length. High-risk preferences require explicit
  `/preferences set` invocation.
- **Don't apply a remembered preference inside this skill.** This
  skill manages preferences and the decision log; it doesn't act
  on them. Other skills read `state/preferences.md`,
  `state/decision-log.md` and apply.
- **Don't auto-commit.** The user reviews and commits.
- **Don't write evidence as a paraphrase.** Quote the user. If you
  can't quote (e.g. the trigger was a `/preferences set` invocation),
  record the invocation as the evidence.
- **Don't fabricate IDs.** If the user's request doesn't map to a
  known preference ID, surface that. Don't invent.
- **Don't move log entries from Captured (archive) back to Active**
  unless the corresponding preference is being revoked. The
  archive is permanent unless the preference re-enters circulation.

---

## When NOT to use this skill

- **Decision about specific work** (which sub-repos to register, what
  to name a feature) — those go in the artifact, not in preferences.
- **Architectural decision** — `/decision` (ADR).
- **Per-repo gotchas** — `<sub>/.claude/shared/notes.md` or
  `state/sub-repos/<name>.md` Notes.
- **Reading another skill's stored state** — preferences only.

---

## What "done" looks like

For each mode:

- **List:** captured preferences AND tracked-but-not-captured log
  entries rendered one-per-line; revoke and clear-log commands
  surfaced.
- **Show:** full preference entry rendered if captured (with
  archive history summary), OR full log history rendered if
  tracked-but-not-captured (with auto-offer status), OR "nothing
  tracked yet" if neither.
- **Set:** preference written to `state/preferences.md` with all
  required fields; if a log entry existed for this ID, it's moved
  to the archive section; user shown the entry; revoke command
  surfaced.
- **Revoke:** preference entry moved from Active to Revoked
  section; the log archive entry is moved back to Active with a
  fresh History list; user shown both.
- **Clear-log:** the fork's Active section in `decision-log.md` is
  reset (tally 0/0, streak 0, history cleared); optionally
  annotated with reason; preferences are unchanged.

In all cases: no auto-commit. The user commits.
