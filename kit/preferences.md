# Operating preferences

How the orchestrator learns what the CTO wants and stops asking
about it. The dual to [`orchestrator-rules.md`](orchestrator-rules.md)
"When in doubt — ask": **when *not* in doubt — act, and remember
what was settled.**

The principle: less friction, more action where confidence is
earned. Every recurring question that's already been answered the
same way is a friction tax with no information value. Capture the
answer once; apply it from then on; surface that you're applying
it so the CTO can audit.

---

## Where it lives

Per-instance state file: **`state/preferences.md`**.

Committed (versioned, visible to collaborators on the same instance).
Not in `local-config.json` — that's per-machine and gitignored. A
preference is the CTO's call about how this orchestrator operates;
it persists across machines and shows up in PRs.

Initial file is bootstrapped from
[`bootstrap/preferences.md.template`](../bootstrap/preferences.md.template)
on `bin/init` (skip-if-exists).

---

## What a preference is

A **preference** is a recorded answer to a recurring decision point
the orchestrator would otherwise ask about.

Properties:

- **id** — slug, stable across the lifetime of the preference
  (e.g. `auto-scaffold-shared-on-register`)
- **decision** — `yes` / `no` / a short value
- **scope** — what the preference applies to (global, per-sub-repo, etc.)
- **tier** — `low-risk` / `high-risk` / `never-suppressible` (see below)
- **set** — date + handle of who set it
- **evidence** — the user statement or signal that established the
  preference. Verbatim quote where possible.
- **revoke** — how to revoke (skill command, edit, etc.)

A preference is **not** a one-off decision about specific work
(those go in feature plans, migrations, ADRs). It's a meta-decision
about how the orchestrator interacts with the user.

---

## Risk tiers

Every preference declares a tier. The tier governs what kind of
signal can establish it.

### Low-risk

The decision is reversible, low blast radius, and the cost of
applying it without re-asking is minimal.

- Examples: scaffold `shared/` on register, install claude-kit on
  non-kit sub-projects, append to `references.md` on discovery,
  default branch convention.
- Capture rule: explicit user statement — "always do X," "yes,
  and don't ask," "stop asking me about X."
- Application: applied silently after first-session disclosure
  (see "Disclosure" below).

### High-risk

The decision can have meaningful consequences if applied wrongly,
even though it's not destructive.

- Examples: auto-merging orchestrator-authored PRs, auto-pushing
  branches to remote, sending external messages, triggering CI/CD
  workflows.
- Capture rule: extra-clear explicit signal — "yes, auto-merge orch
  PRs going forward, I'm sure" — and the orchestrator confirms
  back: "recording high-risk preference `<id>: <value>`. Revoke
  with `/preferences revoke <id>`."
- Application: applied with first-session disclosure, plus
  surface in `/status` so the CTO sees high-risk preferences in
  every status view.

### Never-suppressible

Cannot be set as a preference. The orchestrator always asks per
action.

- Examples: code-boundary override (the "touch a running file"
  override per `sub-projects.md`), destructive git ops (force-push
  to main, branch deletion of unmerged work), preference revocation
  itself.
- Capture rule: refused. If user says "always override the code
  boundary," the orchestrator declines and explains why.

---

## Capture protocol

A preference is captured **only** on explicit user signal. v1 does
no inference from repeated yes-answers.

### Signals that capture

- "always do X" / "always X"
- "from now on, X"
- "stop asking me about X" / "stop asking, just do it"
- "remember this — X"
- "yes, and don't ask again"
- Direct invocation: "/preferences set `<id>` `<value>`"

### When the orchestrator should ask "want me to remember?"

After answering a question that's preference-eligible (the question
maps to a known preference ID and is in low-risk or high-risk tier),
the orchestrator MAY append:

> "Want me to remember this for next time? Save preference
> `<id>: <value>`?"

This is **opt-in capture** — one extra short question, zero
hidden capture. If the user says no, no preference is set.

For high-risk tier, the prompt is more explicit:

> "Want me to remember this and apply it automatically going
> forward? `<id>` is high-risk — I'll surface it in `/status`
> so it stays visible. Save? [yes / no]"

### What captures

When the user signals capture, write the preference to
`state/preferences.md` with all required fields populated. Tell
the user what was written, including the revoke command.

---

## Application protocol

When a skill hits a fork that maps to a known preference ID:

1. **Look up the preference** in `state/preferences.md`.
2. **If absent**: ask the question normally; offer capture per
   above.
3. **If present**: apply the preference's decision and skip the
   question. **Disclose on first apply per session** (see below).

### Disclosure rule

The first time a skill applies a remembered preference in a session,
surface it in the chat output:

```
ℹ Per preference `auto-scaffold-shared-on-register` (set 2026-05-08
  by chazz: "always scaffold"), scaffolding shared/ automatically.
  Revoke: `/preferences revoke auto-scaffold-shared-on-register`
```

Subsequent applies of the same preference in the same session: silent.
The point of disclosure is auditability without cluttering output.

For high-risk preferences, surface every time, not just once per
session.

### When to refuse application

- The preference is set but stale (set >180 days ago and has not
  been applied or re-confirmed since): apply but disclose with a
  staleness flag, suggest the CTO confirm or revoke.
- The preference exists but the current context differs in a way
  that matters (e.g. the preference says "auto-scaffold for sub-repos"
  but the sub-repo in question is unusual — different parent org,
  larger blast radius). The skill author decides per-skill what
  warrants re-asking. **When unsure, ask.**

---

## Revocation

Three paths:

1. **`/preferences revoke <id>`** — explicit. Confirm + delete the entry.
2. **Edit `state/preferences.md` directly** — for batch cleanup. Same effect.
3. **Implicit revocation by user statement** — "stop auto-scaffolding"
   → `/preferences revoke auto-scaffold-shared-on-register`. The
   skill recognizes the revoke pattern and confirms before deleting.

Revocation is logged in the file (the entry isn't fully deleted —
its body is replaced with a "revoked" marker plus the revoke date
and reason). Audit trail.

---

## Known preferences

Skill authors register preference IDs here when they add a fork that
should be preference-eligible. Skills check the registry; the
registry is the contract.

| ID | Tier | Owning skill(s) | What it controls |
|---|---|---|---|
| `auto-scaffold-shared-on-register` | low-risk | `/register` | Whether `<sub>/.claude/shared/` is scaffolded automatically on new sub-repo registration without asking. |
| `auto-install-kit-on-non-kit-register` | low-risk | `/register` | Whether claude-kit is installed automatically on registration of a non-kit sub-project (when the offer would otherwise be made once). |
| `auto-write-active-features-notices` | low-risk | `/feature` | Whether per-affected-sub-repo `active-features.md` notices are written without per-call confirmation. |
| `auto-write-active-migrations-notices` | low-risk | `/migration` | Whether per-affected-sub-repo `active-migrations.md` notices are written without per-call confirmation. |
| `auto-merge-orch-prs` | high-risk | any skill that opens a `chore/orch-*` PR | Whether orchestrator-authored PRs in sub-repos are auto-merged after open (vs. left for user merge). **Default: no.** |
| `auto-push-orch-branches` | high-risk | any skill that creates `chore/orch-*` branches | Whether orchestrator-created branches are auto-pushed to origin (vs. left local for user push). |

Adding a new preference: append to this table, declare its tier,
update the owning skill(s) to check `state/preferences.md` and
respect the preference, document the trigger phrases the
`/preferences` skill should recognize.

---

## What this is NOT

- **Not** company-level config (that's `company-profile.md`,
  `tech-principles.md`, etc.).
- **Not** per-repo gotchas (those go in
  `state/sub-repos/<name>.md` Notes or `<sub>/.claude/shared/notes.md`).
- **Not** ADRs (those are architectural calls, not operational
  preferences).
- **Not** memory in the sense of "Claude's running notes" —
  preferences are deliberate, durable, audit-able decisions about
  orchestrator behavior. Read `MEMORY.md` for the running-notes
  side.

---

## See also

- [`skills/preferences/SKILL.md`](skills/preferences/SKILL.md) —
  the user-facing skill for inspect/set/revoke
- [`orchestrator-rules.md`](orchestrator-rules.md) "When in doubt"
  — the complementary rule for the asking side
- [`bootstrap/preferences.md.template`](../bootstrap/preferences.md.template)
  — the initial-state file template
