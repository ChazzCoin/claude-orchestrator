# Sub-repo shared context

Templates for the **durable, two-way per-repo memory** at
`<sub-repo>/.claude/shared/`. Both the orchestrator and the sub-kit
(and humans) read and write these files. They are **not**
auto-managed and **not** regenerated.

This is the sibling discipline to [`sub-repo-notices/`](../sub-repo-notices/),
not a replacement. They serve different purposes:

| | `sub-repo-notices/` | `sub-repo-shared/` (this) |
|---|---|---|
| **File pattern** | `<sub>/.claude/active-<concern>.md` | `<sub>/.claude/shared/<topic>.md` |
| **Authoring** | Auto-managed by an owning skill | Hand-edited (orchestrator skills, sub-kit, or humans) |
| **Lifecycle** | Regenerated wholesale on each write; deleted on empty | Append-only or edit-in-place; persists indefinitely |
| **Direction** | Orchestrator → sub-repo | Bidirectional |
| **Use** | Current-state signals (open migrations, etc.) | Accumulated context (architecture, notes, message threads) |
| **Discipline owner** | One skill per file | Convention; whoever's working in this repo |

Mixing the two patterns is the failure mode this split prevents:
auto-regenerated content kills hand edits; hand-edited content
diverges from the auto-managed view of state. Keep them separate.

## Files in this template set

| File | Audience | What goes in it |
|---|---|---|
| [`architecture.md.template`](architecture.md.template) | dev + orchestrator | Living doc of how this repo is laid out — modules, data flow, key decisions, gotchas. Updated when architecture changes. |
| [`inbox.md.template`](inbox.md.template) | dev + orchestrator | Append-only message thread for this repo. Both directions write. Different from per-person `/inbox`. |
| [`notes.md.template`](notes.md.template) | dev + orchestrator | Freeform shared memory. Things to remember about this specific repo that don't fit elsewhere. |
| [`references.md.template`](references.md.template) | dev + orchestrator | Curated URLs to official docs, repos, dashboards relevant to this codebase. The "where do I look for X" file. |

## When the orchestrator writes here

Specific cases the orchestrator may write into `shared/*.md`:

- **`shared/inbox.md`** — when the orchestrator wants to leave a
  durable note for this repo's next session ("contracts/events.md
  changed, you'll want to update your handler"). Append-only.
- **`shared/architecture.md`** — only on explicit user direction
  ("the orchestrator should record that `X` lives in `Y`"). Default
  authorship is the dev or sub-kit, not the orchestrator. The
  orchestrator's role is mostly *reading* this file when authoring
  features, not writing it.
- **`shared/references.md`** — when the orchestrator discovers a doc
  URL during work that should live with this repo. Append.
- **`shared/notes.md`** — append for durable observations the
  orchestrator wants the next session to see.

The orchestrator follows the standard write protocol from
[`sub-projects.md`](../../sub-projects.md):

1. Pull latest main in the sub-repo
2. Branch: `chore/orch-<YYYY-MM-DD-slug>`
3. Write the change (append or edit)
4. Open PR titled `orch: <one-line summary>`
5. User approves the merge

These are non-running files, so no code-boundary concerns.

## When the sub-kit writes here

The sub-kit may write to `shared/` whenever the dev (or claude on
the dev's behalf) decides durable context is worth preserving. Same
file pattern, no special protocol — the sub-kit owns its own repo
and does normal git work.

The discipline expectation: **append, don't churn**. These files
exist to accumulate context, not to be rewritten each session.

## Bootstrap (creating these files in a new sub-repo)

When a sub-repo is registered with the orchestrator via `/register`,
the skill **offers** to scaffold `<sub>/.claude/shared/` (step 11 of
[`kit/skills/register/SKILL.md`](../../skills/register/SKILL.md)).
The user accepts or declines:

- **Accept** → the orchestrator opens a `chore/orch-init-shared` PR
  in the sub-repo with the four files rendered from these templates.
  User merges when ready.
- **Decline** → no files are written. The channel can be created
  lazily later (the first time something is written into one of the
  `shared/*.md` files).

`bin/setup` (collaborator clone flow) does **not** scaffold — by the
time a collaborator clones a sub-repo, the `shared/` directory is
either already in main (because the original registrar accepted the
scaffold offer) or it doesn't exist yet (and lazy creation will
handle it).

### Retroactive scaffolding (already-registered sub-repos)

For sub-repos registered before this channel existed, run the same
recipe by hand:

1. In `repos/<name>/`, branch from `main`: `git checkout -b chore/orch-init-shared`
2. Render the four templates with `{{REPO_NAME}}`, `{{TIMESTAMP}}`,
   `{{DATE}}` substituted; write them into `.claude/shared/`.
3. Commit with subject `orch: scaffold .claude/shared/ for durable two-way context`.
4. Push and `gh pr create` with a body linking back to
   [`kit/templates/sub-repo-shared/README.md`](.) and
   [`kit/governance/dev-preflight.md`](../../governance/dev-preflight.md).
5. User merges.

Or ask Claude in the orchestrator to "apply the `/register` step 11
recipe to `<name>`" — same thing, less typing.

## File-on-disk naming

```
<sub-repo>/.claude/shared/architecture.md
<sub-repo>/.claude/shared/inbox.md
<sub-repo>/.claude/shared/notes.md
<sub-repo>/.claude/shared/references.md
```

Flat, no subdirectories. If a sub-repo grows beyond these four
topics (project-specific concerns), add new files at the same
level — but only after deciding they don't belong in `notes.md`
or as a new `active-<concern>.md` instead.

## What you must NOT do

- **Don't auto-regenerate `shared/*` files.** That breaks the
  hand-edit contract and discards user content.
- **Don't write the same content in `shared/` and in
  `active-<concern>.md`.** Pick one based on lifecycle: regenerated
  → notices, durable → shared.
- **Don't put time-sensitive state in `shared/`.** If it goes stale
  fast (migration status, current branch, etc.), it belongs in
  `active-<concern>.md` or `state/sub-repos/<name>.md`, not here.
- **Don't write secrets or PII into `shared/*`.** These files are
  in the sub-repo and likely reviewed across the team. Same hygiene
  as any committed file.

## Adding a new shared topic (future)

If a new shared-context file makes sense (e.g., `shared/glossary.md`
for project-specific terminology), follow the pattern:

1. Add a template here at `kit/templates/sub-repo-shared/<topic>.md.template`.
2. Update this README's table.
3. Decide whether the orchestrator should reference or write to the
   new file in any of its skills. Update the relevant skills.
4. The convention stays: durable, hand-edited, append-friendly.
