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

Two paths into a preference:

1. **Explicit signal** — the user says "always do X." Captured
   immediately.
2. **Confidence-driven offer** — the orchestrator has observed N
   consecutive same-answer responses to a fork (logged in
   `state/decision-log.md`) and offers capture. User accepts or
   declines per-offer.

Both paths produce the same preference entry; the difference is
how the orchestrator was prompted to capture.

### Path 1: Explicit signal

Signals that capture immediately:

- "always do X" / "always X"
- "from now on, X"
- "stop asking me about X" / "stop asking, just do it"
- "remember this — X"
- "yes, and don't ask again"
- Direct invocation: "/preferences set `<id>` `<value>`"

When the user signals capture, write the preference to
`state/preferences.md` with all required fields populated. Tell
the user what was written, including the revoke command.

### Path 2: Confidence-driven offer

Each time a skill asks a known-fork question and gets an answer,
the skill logs the decision in
[`state/decision-log.md`](../state/decision-log.md) per the
format spec there.

After logging, the skill checks the log's aggregate for the fork:

- **If the streak of consecutive same-answers ≥ 3 AND the fork is
  low-risk AND no offer cooldown is active**, the skill offers:

  > "I've seen you answer `<value>` to this <N> times in a row.
  > Want me to remember it as a preference and stop asking?
  > [yes / no]"

  - **yes** → capture as a preference (path 1's effect). The fork
    moves to `state/preferences.md`; future invocations apply
    silently with disclosure. Decision log entries for this fork
    are pruned (kept as audit, but the active counter resets).
  - **no** (or silence) → no capture. Set a cooldown: the offer
    won't repeat until 5 more decisions have been logged for this
    fork. The decline itself is logged.

- **If the fork is high-risk**, no auto-offer is made regardless
  of streak length. High-risk preferences always require explicit
  `/preferences set` invocation. The decision log still tracks
  high-risk forks (for audit), but the orchestrator does not
  prompt for capture.

- **If the streak < 3 OR a cooldown is active**, no offer. Just
  log and move on.

### What gets logged

Only known preference IDs (per the registry below). Skills that
hit a fork without a registered preference ID don't log — there's
nothing to aggregate. To make a new fork preference-eligible,
register the ID in the table at the bottom of this file.

---

## Skill recipe at a known fork

The exact algorithm a skill follows when it hits a question that
maps to a registered preference ID. **Every preferences-aware skill
follows this five-step recipe.** It is the contract that makes the
preferences + decision-log system actually consumable.

### The five steps

1. **Read `state/preferences.md`.** Look up the preference ID.
   - **If found in `## Active preferences`** → apply the recorded
     decision silently. **Disclose on first apply per session** per
     "Application protocol" above. Skip steps 2–5.
   - **If found in `## Revoked preferences`** → ignore the
     revocation; the fork is back to "ask normally." Proceed to
     step 2.
   - **If not found** → proceed to step 2.

2. **Ask the question** normally.

3. **After receiving the answer**, apply it (the action the
   question gates).

4. **Log the decision** to `state/decision-log.md` per "Decision
   log discipline" above:
   - Locate or create the fork's `## <id>` section under `## Active`.
   - Prepend a new entry to the History list:
     `- <YYYY-MM-DD> <handle>: <value> [— <context>]`
   - Recompute the aggregate header (tally, streak, last asked).
   - Decrement the cooldown counter if active; expire it at 0.

5. **Check the offer threshold** for this fork:
   - **High-risk tier:** never auto-offer regardless of streak.
     Skill stops here. The user can still set the preference
     explicitly via `/preferences set`.
   - **Low-risk tier, streak ≥ 3, no active cooldown:** prompt:
     > "I've seen you answer `<value>` to this <N> times in a row.
     > Want me to remember it as a preference and stop asking?
     > [yes / no]"
     - **yes** → write the preference entry to
       `state/preferences.md` (evidence: the user's most recent
       answer + this offer-acceptance), move the fork's section in
       the decision log from `## Active` to `## Captured (archive)`,
       disclose the capture.
     - **no / silence** → update the fork's `Last offer:` field in
       the log to `<YYYY-MM-DD> declined (cooldown: 5 decisions remaining)`.
       Continue.
   - **Otherwise** (streak < 3 or cooldown active) → stop. No
     offer.

### What "preferences-aware" means in a skill's SKILL.md

A skill that asks at least one known-fork question is
preferences-aware. Its SKILL.md must:

- Add a bullet to its **Behavior contract** naming the fork(s) it
  asks and the preference ID(s) registered for them.
- Reference this recipe so the skill's prompts implement steps
  1–5 at the right point in the flow.
- Not invent its own capture, application, or threshold logic.
  The recipe is the contract.

### What "preferences-aware" does NOT mean

- It does **not** mean every prompt is a preference fork. Skills
  ask many one-off questions (which sub-repo, what slug, etc.)
  that don't generalize and aren't logged.
- It does **not** mean the skill can write preference entries on
  its own bypass of `/preferences`. Path 1 (explicit signal) and
  path 2 (offer accepted) both go through the same write path —
  the offer's "yes" branch is the only programmatic write
  preference-aware skills perform.

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

After revocation, the decision log resumes accumulating for that
fork — the orchestrator goes back to asking, and may eventually
offer capture again if a new streak forms.

---

## Decision log discipline

### File location

`state/decision-log.md`. Bootstrapped from
[`bootstrap/decision-log.md.template`](../bootstrap/decision-log.md.template)
(skip-if-exists).

### Who writes

- **Skills with known forks** write entries directly when they ask
  a known-fork question and get an answer. Format spec in the
  template; one-line entries with optional context.
- **`/preferences`** reads (for `list` and `show` modes) and prunes
  (when a preference is captured, the active counters reset for
  that fork). Doesn't otherwise write.

### What's tracked per fork

A `## <preference-id>` section per fork that's been asked at least
once. Section header carries the aggregate; entries below it are
the rolling history.

```markdown
## auto-scaffold-shared-on-register

- **Tally:** yes 5 / no 0 (streak: 5 yes)
- **Last asked:** 2026-05-09
- **Last offer:** never (or `<YYYY-MM-DD> declined`, with cooldown counter)
- **Tier:** low-risk

### History

- 2026-05-09 chazz: yes — sub-repo: ios
- 2026-05-08 chazz: yes — sub-repo: web
- 2026-05-07 chazz: yes — sub-repo: api
- 2026-05-05 chazz: yes — sub-repo: devops
- 2026-05-03 chazz: yes — sub-repo: payments
```

History is **prepend** (newest first). Aggregate header is
**recomputed on each write**:

- `yes` count = number of yes entries in History
- `no` count = number of no entries in History
- `streak` = run length of consecutive same-answer entries from the
  top of History
- `last asked` = top entry's date
- `last offer` = updated when an auto-offer is made; carries the
  outcome (`accepted` and the entry is then pruned from log; or
  `declined` with cooldown counter)
- `tier` = mirrored from the registry below; informational

### How a skill writes a log entry

1. Read `state/decision-log.md`. Locate or create the `## <id>`
   section.
2. Prepend a new line to the History list:

   ```
   - <YYYY-MM-DD> <handle>: <value> [— <context>]
   ```

   Context is optional; useful when the answer applies to a
   specific sub-repo or artifact.
3. Recompute the aggregate header (yes/no/streak/last asked).
4. Write back.

The format is plain markdown. Skills without a shared helper still
produce consistent files as long as they follow the spec exactly.

### Cooldown bookkeeping

When an auto-offer is **declined**, set:

```
- **Last offer:** <YYYY-MM-DD> declined (cooldown: 5 decisions remaining)
```

On each subsequent log entry for the same fork, decrement the
cooldown counter. When it reaches 0, the field becomes
`<YYYY-MM-DD> declined (cooldown expired)` and offers are eligible
again at the next streak threshold.

### Pruning

When a preference is captured (path 1 explicit OR path 2 accepted):

- The fork's active section in `decision-log.md` is **moved** to a
  `## Captured (archive)` section at the bottom of the file. The
  full history is preserved for audit; the active counter is no
  longer maintained for that fork.
- If the preference is later revoked, the fork's section is moved
  back to active and a fresh `### History` list begins (the
  archive remains intact).

### What this is NOT

- **Not** a general-purpose user activity log. Only known
  preference IDs are tracked.
- **Not** a substitute for the Audit log. `AUDIT.md` is the
  chronological record of significant orchestrator actions; the
  decision log is per-fork tally only.
- **Not** machine learning or inference. The threshold and tier
  rules are simple, deterministic, and documented.

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
  the user-facing skill for inspect/set/revoke + log read
- [`orchestrator-rules.md`](orchestrator-rules.md) "When in doubt"
  / "When *not* in doubt" — the complementary rules for the asking
  and acting sides
- [`bootstrap/preferences.md.template`](../bootstrap/preferences.md.template)
  — the initial preferences file
- [`bootstrap/decision-log.md.template`](../bootstrap/decision-log.md.template)
  — the initial decision-log file
