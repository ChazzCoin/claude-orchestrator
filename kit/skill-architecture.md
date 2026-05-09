# Skill architecture — when to script, when to interpret

How orchestrator skills are built. Generic across all instances —
synced via `/sync` (kit-managed).

The orchestrator's skills come in **two shapes**, and the shape is
decided by what the flow needs to do — not by historical accident.

## Why this matters

LLMs are excellent at interpretation, drafting, and judgment. They
are **worse than computers** at predictable, deterministic behavior
— same input, same output, every time. For mechanical processes
(fetching git refs, parsing manifest entries, writing state files),
interpretation isn't a feature — it's a leak. Each "interpretation"
is a chance for the flow to drift.

Three concrete benefits of scripting the mechanical parts:

1. **Lockdown.** The flow is the script. No skills-doc drift over
   time, no "this run vs that run" variance. The behavior is
   reproducible by definition.
2. **Documentation by code.** The script *is* the spec. Reading
   `bin/refresh` tells you exactly what `/refresh` does — no
   gap between intent and implementation.
3. **User-runnable.** `bin/refresh` works from a terminal without
   Claude. The skill is a convenience layer for people working
   inside Claude Code; the script is the engine.

## The two shapes

### Scripted skills

The flow is a defined sequence of mechanical steps. The skill is a
**thin Claude wrapper** that invokes a shell script (typically at
`bin/<name>`) and surfaces output. Behavior is locked, identical run
to run, no interpretation drift.

The skill markdown is short — typically <50 lines. It says: "run the
script, show the output, don't reinterpret."

Examples:

- `/refresh` → `bin/refresh`
- `bin/setup` (no skill — pure script, run by collaborator)
- `bin/init` (no skill — kit-side, runs against instances)

### Interpretive skills

The flow requires drafting, summarization, conversation, or judgment
that Claude is uniquely good at. The skill is a markdown contract
Claude follows; there's no script.

Examples: `/decision`, `/feature`, `/onboard`, `/audit`,
`/incident`.

### Mixed skills

The flow has a mechanical part (Claude shouldn't redo it each run)
plus a conversational part that needs Claude's interpretation. The
skill calls scripts for mechanical bits and drives conversation
around them.

Examples:

- `/register` — script verifies + clones; skill handles Q&A.
- `/migration` — script creates files; skill drafts content.

The split-point is the skill's choice; document it in the SKILL.md.

## Decision rule — when to script

Convert a skill to scripted form when **all** are true:

- The flow has clearly defined mechanical steps; no conversation.
- The output is data (JSON, markdown updates, file writes), not
  prose drafted from context.
- Variance between "good runs" is zero; only environmental
  failures (auth, network, missing files) cause variation.
- The skill currently spends most of its tokens describing how to
  do things rather than what to draft.

If any of those is false, keep the skill interpretive. Don't force
mechanical structure onto creative work.

## Configuration: scripts read from `.claude/local-config.json`

Per-machine values (identity, paths, secret-pointers, preferences)
live in `.claude/local-config.json` (gitignored, set by `bin/setup`
on first run). Scripts read fields as needed — turning the file into
a per-machine data model that scripts treat as arguments.

Example fields the schema can grow to support:

```json
{
  "schema_version": 1,
  "me": "chazz",
  "machine_id": "chazz-laptop",
  "inbox_last_read": "2026-05-08T14:30:00Z",
  "secrets": {
    "github_token_path": "/Users/chazzromeo/.config/gh/token",
    "aws_profile": "rai-prod"
  },
  "preferences": {
    "default_pr_branch_prefix": "chore/orch-",
    "fetch_timeout_seconds": 30
  }
}
```

`me`, `machine_id`, `inbox_last_read` are the v1 fields populated by
`bin/setup`. `secrets.*` and `preferences.*` are extension points
for scripts that need them — populated by hand or by future targeted
scripts.

Scripts read fields with python (stdlib only — no jq dependency):

```bash
LOCAL_CONFIG="$ORCH_ROOT/.claude/local-config.json"
ME=$(python3 -c "import json; print(json.load(open('$LOCAL_CONFIG'))['me'])")
```

With a default fallback:

```bash
GH_TOKEN_PATH=$(python3 -c "
import json
d = json.load(open('$LOCAL_CONFIG'))
print(d.get('secrets', {}).get('github_token_path', ''))
")
```

When a script needs a field that doesn't exist, it errors clearly
with the field name and a suggestion: `add 'secrets.aws_profile' to
.claude/local-config.json`.

## Templates for instance-specific scripts

When a script's flow is generic but specific commands depend on the
instance (a deploy script's exact AWS region, IAM role, image tag
strategy), ship it as `bootstrap/bin/<name>.sh.template` with
comments showing what to fill in. Treat it like the bootstrap
markdown templates — copied at init, becomes instance property, not
overwritten by `/sync`.

Two layers of scripts result:

| Layer | Where in kit | Where in instance | Synced by |
|---|---|---|---|
| Generic | `bin/<name>` | `bin/<name>` | `/sync` (auto-updates) |
| Templated | `bootstrap/bin/<name>.sh.template` | `bin/<name>.sh` | init only (skip-if-exists) |

Generic scripts are shared discipline — improvements propagate to
every instance. Templated scripts are instance-owned — once filed in,
the instance's version diverges from the template intentionally.

## File-layout conventions

| What | Where |
|---|---|
| Generic user-runnable scripts | `bin/<name>` (kit-synced, executable) |
| Templated scripts | `bootstrap/bin/<name>.sh.template` |
| Skill markdown | `kit/skills/<name>/SKILL.md` |
| Per-machine config | `.claude/local-config.json` (gitignored) |
| Locked flow output | `state/<name>.json` or `state/<name>.md` |

When a skill is purely scripted, `kit/skills/<name>/SKILL.md` is
short and points at `bin/<name>`. When a skill is mixed, the
SKILL.md describes which steps invoke which script.

## Output discipline — pick a pattern

Every script's terminal output picks a pattern from
[`output-catalogue.md`](output-catalogue.md) — the kit-managed
visual design catalogue. Don't invent custom output shapes.
Consistency across all orchestrator scripts means a reader who's
learned one output knows how to read every other.

The catalogue contains 34 patterns covering completion cards,
status dashboards, roadmaps, audits, deployment reports, alerts,
empty states, and more. High-relevance patterns for the
orchestrator domain are listed in the catalogue's intro.

### Two requirements for any new script

1. **Declare the pattern in the script header.** The first comment
   block of the script names the catalogue pattern its output uses:

   ```bash
   #!/usr/bin/env bash
   #
   # bin/<name> — <one-line purpose>
   #
   # Output: Pattern 17 — Git branch overview
   #         (per-repo state with status indicators)
   #
   # ... rest of header
   ```

2. **Reference the pattern in the SKILL.md.** When the skill
   describes what the user sees, it cites the pattern by number and
   name so the catalogue is the spec.

### When the catalogue doesn't have a pattern that fits

Two paths, in order of preference:

1. **Compose existing patterns.** Most outputs are a section banner
   (10) plus a status list (17 / 28 / 22). Combining is fine; the
   header just declares the composition.
2. **Propose a new pattern.** If a genuinely new shape is needed,
   propose it as an addition to `output-catalogue.md` (PR), then
   use it once it's accepted. Don't ship a one-off shape outside
   the catalogue — that defeats the discipline.

### Output discipline applies to interpretive skills too

When an interpretive skill (`/decision`, `/feature`, etc.) writes
to stdout for the user — summary, drafts, confirmations — the
SKILL.md should still cite a pattern from the catalogue. The
output may not be byte-identical run to run (it's drafted), but
the *shape* (banner + list, alert variant, comparison matrix)
should be consistent.

## What NOT to script

- **Conversational drafting** — `/decision`, `/feature`, `/onboard`,
  `/audit`. These benefit from Claude's interpretation; locking
  them down removes value.
- **Truly variable flows** — if steps depend on context Claude alone
  can read (e.g., "draft a postmortem from this incident's facts"),
  keep it interpretive.
- **One-off operations** — if you'll only ever run it once,
  scripting is overhead. Just have Claude do it.

## When in doubt

Ask: *can a future maintainer reproduce the exact behavior of this
skill by reading a script?* If yes, script it. If no, the value is
in Claude's judgment — keep it interpretive.

## Migration path — converting an existing skill

When converting an interpretive skill to scripted form:

1. Identify the deterministic core. What ARE the mechanical steps?
2. Write the script. Test it standalone (it should work without
   Claude in the loop).
3. Rewrite the SKILL.md as a thin wrapper. Keep only what Claude
   needs to know: which script to call, what to surface from
   output, what NOT to do (don't reimplement, don't bypass).
4. Wire MANIFEST.json to sync the script. Wire `bin/init` to copy
   it into instances at bootstrap time.
5. `/sync` rolls the change into existing instances.

The first conversion is `/refresh` (see `bin/refresh` and
`kit/skills/refresh/SKILL.md`). It's a clean prototype to follow.
