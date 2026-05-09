---
name: preferences
description: Inspect, set, or revoke operating preferences — recurring decisions the orchestrator has learned and no longer asks about. Triggered by "/preferences", "/prefs", "what have you remembered", "show my preferences", "stop asking me about X", "always do X", "from now on X", "remember to X", "forget the preference about X", "revoke preference X".
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
  preferences" → enumerate all active preferences (and optionally
  revoked).
- **Show** — "/preferences show `<id>`", "tell me about preference X"
  → show one preference's full entry including evidence.
- **Set** — "always do X", "from now on X", "stop asking about X",
  "remember this", "/preferences set `<id>` `<value>`" → record
  a preference.
- **Revoke** — "stop auto-X", "forget the preference about X",
  "/preferences revoke `<id>`" → revoke a preference.

If ambiguous, ask which.

---

## File location and format

Active preferences live at `state/preferences.md`. Format
documented in [`.claude/preferences.md`](../../preferences.md)
"What a preference is" + the template at
`bootstrap/preferences.md.template`.

This skill is the **only** thing that should write to that file.
Hand edits are tolerated but the file is rewritten alphabetically
on each skill write.

---

## Mode: List

### Process

1. Read `state/preferences.md`. If absent, surface "no preferences
   yet — the orchestrator hasn't learned anything to skip" and
   stop.
2. Parse `## Active preferences` section into entries.
3. Render compact:

   ```
   active preferences (N):

     auto-scaffold-shared-on-register   yes  · low-risk    · 2026-05-08 chazz
     auto-install-kit-on-non-kit-register yes · low-risk    · 2026-05-08 chazz
     auto-merge-orch-prs                no   · high-risk   · 2026-05-09 chazz

   high-risk preferences are also surfaced in /status.
   revoke any with: /preferences revoke <id>
   ```

4. If user asked for revoked too ("/preferences list all" or "show
   revoked"), parse the revoked section and append:

   ```
   revoked (N):
     auto-push-orch-branches  was: yes  · revoked 2026-05-10 chazz · "too risky"
   ```

### Style

- One line per preference. ID, decision, tier, set date, who.
- Tier shown explicitly so high-risk is visible at a glance.
- Don't show evidence or full body in list mode — that's `show`.

---

## Mode: Show

### Process

1. Identify the `<id>`. If user said "tell me about auto-scaffolding,"
   match against the known IDs from
   [`.claude/preferences.md`](../../preferences.md) "Known
   preferences" table; if ambiguous, list candidates.
2. Read the full entry from `state/preferences.md`.
3. Render the full entry: id, decision, scope, tier, set, evidence
   verbatim, revoke command.
4. If the preference is **stale** (set >180 days ago), flag it and
   suggest re-confirm or revoke.

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
   > orchestrator will start asking again. [yes / no]"

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

6. Tell the user. Show the revoked entry.

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

- **Don't infer preferences from repeated yes-answers.** v1 captures
  only on explicit signal. Adding inference-based capture is a v2
  decision.
- **Don't capture preferences for `never-suppressible` IDs.** Even
  if the user insists. Explain why and stop.
- **Don't apply a remembered preference inside this skill.** This
  skill manages preferences; it doesn't act on them. Other skills
  read `state/preferences.md` and apply.
- **Don't auto-commit.** The user reviews and commits.
- **Don't write evidence as a paraphrase.** Quote the user. If you
  can't quote (e.g. the trigger was a `/preferences set` invocation),
  record the invocation as the evidence.
- **Don't fabricate IDs.** If the user's request doesn't map to a
  known preference ID, surface that. Don't invent.

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

- **List:** active (and optionally revoked) preferences rendered
  one-per-line; revoke command surfaced.
- **Show:** full entry rendered; staleness flag if applicable.
- **Set:** preference written to `state/preferences.md` with all
  required fields; user shown the entry; revoke command surfaced.
- **Revoke:** entry moved from Active to Revoked section with
  reason (if given); user shown the moved entry.

In all cases: no auto-commit. The user commits.
