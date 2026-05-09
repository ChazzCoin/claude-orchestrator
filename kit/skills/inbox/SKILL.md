---
name: inbox
description: Per-person messaging inside the orchestrator. Send a message to a teammate (lands in their inbox file, committed via git, read on their next pull) or read messages addressed to you. Triggered by "/inbox", "send andrew a message", "any messages for me", "what's in my inbox", "tell andrew X", "did andrew see my message".
---

# /inbox — Per-person messaging

Send and read addressed messages inside the orchestrator. Messages
live at `state/inbox/<person>.md` (one file per recipient). The
recipient sees their unread messages when they run `/inbox`, and
unread count surfaces in `/status`.

This is **in-orchestrator messaging**, not email or Slack. Use it
for things tied to orchestrator context — "I changed the api roadmap,
take a look," "we should talk about the events.md entry for Friday."

## Identity

Every machine has `.claude/local-config.json` (gitignored) with:

```json
{
  "schema_version": 1,
  "me": "<handle>",
  "machine_id": "<machine-name>",
  "inbox_last_read": "<ISO-8601 or null>"
}
```

`me` is set during `bin/setup`. That's how this skill knows who's
asking. If `local-config.json` is missing, abort and tell the user
to run `bin/setup` (which will prompt once for identity).

## Modes

Four modes — detect from user input. If ambiguous, ask.

- **Read** — "/inbox", "any messages for me", "what's in my inbox" →
  show unread for `me`; mark them read after display.
- **Send** — "send andrew a message", "/inbox to:andrew", "tell
  andrew X" → write to `state/inbox/andrew.md`.
- **Sent** — "/inbox sent", "did andrew see my message" → show
  messages `me` has sent recently.
- **List** — "/inbox list", "who has unread" → enumerate all
  inbox files and rough counts.

## Storage

`state/inbox/<recipient>.md` — append-only entries:

```markdown
## 2026-05-08T14:30:00Z — chazz — quick thought on rai-ios deploy

Body markdown here. Code blocks, links, lists all fine.

---
```

Heading: `## <ISO-8601-UTC> — <from-handle> — <subject>`.
Body: freeform markdown until the `---` separator.

Read state is **per-machine** — `inbox_last_read` in
`.claude/local-config.json`. An entry is unread on this machine if
its timestamp > `inbox_last_read`.

## Mode: Read

1. Read `.claude/local-config.json` — get `me` and
   `inbox_last_read` (null = never read anything).
2. Open `state/inbox/<me>.md`. If absent, "no inbox yet — your
   handle is `<me>`."
3. Parse entries by `## ` headings; extract timestamp + from +
   subject + body.
4. Filter to entries with timestamp > `inbox_last_read`.
5. Display oldest-first, compact:

   ```
   ═══ N unread messages for <me> ═══

   ── from chazz · 2026-05-08T14:30 · subject ──
   body

   ── from andrew · 2026-05-08T15:00 · subject ──
   body
   ```

6. Update `inbox_last_read` in `local-config.json` to the latest
   timestamp shown.
7. If no unread → "inbox clear."

## Mode: Send

1. Read `me` from local-config.
2. Identify recipient. If user said "tell andrew", recipient =
   `andrew`. If unclear, ask. If recipient = `me`, confirm
   (self-notes are allowed).
3. Get subject (one-line) and body (markdown) from the user.
   Confirm both before writing.
4. Append to `state/inbox/<recipient>.md`, creating if absent:

   ```
   ## <now-ISO-8601-UTC> — <me> — <subject>

   <body>

   ---
   ```

5. Don't auto-commit. Tell the user the file changed; they commit +
   push. Recipient picks it up after their next `git pull` of the
   orchestrator.

## Mode: Sent

1. Get `me` from local-config.
2. Across all `state/inbox/*.md` files, find entries where
   `<from-handle>` = `me`.
3. Render most-recent-first: `<recipient> · <timestamp> · subject`.
4. Caveat: read state is per-machine and gitignored, so "did they
   see it" isn't knowable from this machine. State that explicitly.

## Mode: List

1. Enumerate `state/inbox/*.md`.
2. For each, count `^## ` headings (rough message count).
3. Display:

   ```
   inbox files:
     chazz    12 entries
     andrew    7 entries
   ```

4. Don't claim unread counts — those are per-machine.

## Style rules

- **Subjects are one line.** Tight.
- **Bodies can be full markdown.** Code blocks, lists, links fine.
- **ISO-8601 UTC timestamps.** No timezone ambiguity.
- **Don't archive on read.** Read just updates `inbox_last_read`;
  entries persist for reference.

## What you must NOT do

- **Don't auto-send.** Always confirm recipient + subject + body
  before writing.
- **Don't commit for the user.** Inbox sends are file changes; user
  reviews and commits.
- **Don't write `inbox_last_read` to committed files.** That value
  lives in gitignored `local-config.json`, never in
  `state/inbox/*.md` or anywhere tracked.
- **Don't deliver outside the orchestrator.** No email, no Slack
  hooks, no notifications. The git pull is the delivery mechanism.

## When NOT to use this skill

- **Announcement for the whole team** — drop a note in `roadmap.md`,
  `AUDIT.md`, or `company-notes.md`. Inbox is for addressed
  messages.
- **Permanent decision record** — that's `/decision`. Inbox messages
  aren't durable artifacts.
- **Time-sensitive calendar item** — that's `events.md`.

## What "done" looks like

- **Read:** unread surfaced; `inbox_last_read` updated in
  local-config.
- **Send:** new entry appended to `state/inbox/<recipient>.md`; user
  notified file changed; commit is on them.
- **Sent / List:** rendered output, no file changes.
