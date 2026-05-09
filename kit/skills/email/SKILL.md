---
name: email
description: Draft (and optionally send) an email to a contact from state/contacts.json. Resolves handles to emails; writes a .eml file in state/drafts/ for audit; sends via SMTP if .claude/integrations-secrets.json is configured. Triggered by "/email", "send andrew an email", "draft an email to lisa", "email <handle> about <topic>", "compose an email", "email update to <handle>".
---

# /email — Draft and send email

Compose an email to one or more contacts from
`state/contacts.json`. The skill is a thin wrapper around
[`bin/send-email`](../../../bin/send-email): it does the Q&A
to gather subject + body + recipients, then invokes the script
which writes a `.eml` draft and (optionally) transmits via SMTP.

For in-orchestrator messaging (per-person inbox files, no real
mail), use [`/inbox`](../inbox/SKILL.md) — different concern,
different audience. `/email` is for actual outbound mail.

**Output pattern:** [Pattern 23 — Activity timeline](../../output-catalogue.md#23--activity-timeline)
for the per-step Q&A flow + [Pattern 1 — Hero completion card](../../output-catalogue.md#1--hero-completion-card)
on draft/send confirmation.

---

## Implementation

This skill is a **thin wrapper** around `bin/send-email`. The
script:

- Reads `state/contacts.json` to resolve handles → emails
- Reads `state/integrations.json` for the from-address (committed)
- Reads `.claude/integrations-secrets.json` for SMTP creds
  (gitignored; only consulted with `--send`)
- Builds RFC 2822 message with proper headers
- Always writes `.eml` to `state/drafts/<timestamp>-<slug>.eml`
  (audit trail)
- With `--send`: also transmits via SMTP

The skill's responsibilities:

- Determine recipients (handles or raw emails)
- Confirm the addressee resolves cleanly via
  `bin/contacts get <handle>` first
- Gather subject + body via Q&A
- Write body to a temp file
- Decide draft-only vs send (default: draft)
- Invoke `bin/send-email` with `--json`
- Render the script's output to the user

### Invocation

```sh
bin/send-email --to <handle-or-email> [--to <another>] \
               [--cc <handle-or-email>] \
               --subject "<subject>" \
               --body-file <path-to-body.md> \
               [--send] [--json]
```

Output (JSON when `--json`):

```json
{
  "ok": true,
  "to": ["andrew@example.com"],
  "to_names": ["Andrew Smith"],
  "subject": "deploy check-in",
  "from": "Chazz Romeo <chazz@example.com>",
  "draft_path": "state/drafts/20260509-120607-deploy-check-in.eml",
  "sent": false
}
```

---

## Behavior contract

- **Resolve recipients via `bin/contacts`.** If the user says
  "email andrew," look up `andrew` in `state/contacts.json`. If
  not found, surface the unknown handle and offer to add the
  contact via `bin/contacts add`. Don't silently treat as a raw
  email unless it contains `@`.
- **Confirm before invoking the script.** Show the user: who it's
  going to (display name + email), the subject, the first ~10
  lines of the body. They confirm; then the script runs.
- **Default to draft-only.** `--send` is opt-in. Reasoning: real
  send requires gitignored credentials; drafts are recoverable;
  the user opens `.eml` in their mail client when ready.
- **Always create a draft, even when sending.** The `.eml` file is
  the audit trail. Sent emails go through `state/drafts/` first.
- **Don't auto-commit.** The user reviews the draft and decides
  what to commit (typically: yes, drafts are committed for audit;
  but not the skill's call).

### Preferences-aware (future)

When this skill matures, the `--send` toggle could become
preference-aware (`auto-send-emails`, low-risk → ask the user
once). Not wired in v1; defer.

---

## Process

### Mode: New email

1. **Identify recipients.**
   - User says "email andrew about deploy."
   - For each name, run `bin/contacts get <handle>` to confirm
     they exist. If not, ask: add now via `bin/contacts add`, use
     a raw email, or cancel?
   - If multiple recipients: ask if any go in CC vs To.

2. **Subject.** One line. Ask if not implied by the user's
   request.

3. **Body.** Either:
   - User provides the full body inline. Skill writes to a temp
     file.
   - Skill drafts a body based on context (recent activity,
     prior emails to this contact, the user's prompt). Show
     draft for confirmation.

4. **Confirm.** Show: To, CC (if any), Subject, body preview
   (first ~10 lines). Ask: "draft only" or "send."

5. **Invoke `bin/send-email`.** Pass `--json`. With `--send` if
   user confirmed real send.

6. **Render result.** Show: draft path, recipient(s), subject,
   sent/drafted status. If sent: confirmation. If drafted: tell
   user how to open the draft (their mail client should
   recognize `.eml`).

### Mode: List drafts

If user asks "what drafts do I have," walk
`state/drafts/*.eml`, parse the From/To/Subject headers, render
a compact list:

```
recent drafts:

  20260509-120607  to: andrew@example.com    subject: deploy check-in
  20260509-093012  to: lisa@example.com      subject: roadmap update
  ...
```

This mode is read-only.

---

## Style rules

- **No marketing voice in drafts.** Match the CTO's tone — direct,
  concise. Don't pad with "I hope this finds you well."
- **One subject line per email.** Don't compose multi-paragraph
  subjects.
- **Short body unless asked otherwise.** Email is for action; if
  a long doc is needed, link to it.
- **Include a clear ask or context.** Every email has either a
  question, a decision needed, or context being shared. If none,
  it's probably a Slack message.

---

## What you must NOT do

- **Don't send without explicit confirmation.** Even if a
  preference is set later, surface what's about to be sent.
- **Don't fabricate email addresses.** Resolve via
  `bin/contacts`. If the contact has no email, surface that.
- **Don't read other people's drafts.** This skill writes to
  `state/drafts/` for the current user; doesn't iterate prior
  drafts unless asked.
- **Don't commit secrets.** SMTP credentials live in
  `.claude/integrations-secrets.json` (gitignored). Don't
  reference them in commits, PRs, or shown output.
- **Don't auto-commit drafts.** User decides.

---

## When NOT to use this skill

- **In-orchestrator messaging** (notes for a teammate who'll pull
  the orchestrator next) → `/inbox`.
- **Repo-bound message** (note for whoever's working in `<repo>`
  next) → append to `repos/<name>/.claude/shared/inbox.md`.
- **Calendar event** → future `/calendar-create` skill (v0.14+).
- **Slack/chat message** → future `/slack` skill.

---

## What "done" looks like

- `state/drafts/<timestamp>-<slug>.eml` written with proper RFC
  2822 headers.
- `bin/send-email`'s JSON output rendered to the user.
- If `--send`: SMTP transmission completed; user shown
  confirmation.
- If draft only: user shown the draft path + reminder how to
  open in their mail client.

In all cases: no auto-commit. User commits the draft (or not).
