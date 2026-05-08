# Sub-kit advertisement template

The **inverse** of `sub-repo-notices/`: instead of the orchestrator
writing into sub-repos, this is the **sub-kit writing to the
orchestrator** — voluntarily — to advertise its current state.

## File on disk

`<sub-repo>/.claude/active.md` (singular, not concern-prefixed) —
because there's only one of these per sub-kit, holding the
sub-kit's overall current state.

## Owning side

**The sub-kit writes; the orchestrator reads.** Reverse of the
`sub-repo-notices/` direction.

The orchestrator reads it via `/sync-check` and surfaces the
contents in:

- `state/sync-status.md` (per-sub-repo snapshot row)
- `state/sub-repos/<name>.md` (the per-sub-repo state file)

## Why voluntary

The orchestrator can infer most of what `active.md` carries from
`git log` + `gh pr list` + `tasks/active/`. The advertisement is
**richer signal** — it carries phase, ETA, and free-form notes
that aren't reconstructable from git.

A sub-kit that doesn't write `active.md` still gets coordinated
with via the inference path. Writing it is a way to say "hey
orchestrator, here's what I know about my own state that you
can't easily figure out from outside."

## When the sub-kit should update it

Per [`sub-projects.md`](../../sub-projects.md) "Sub-kit
advertisement protocol":

- At task start (filing TASK-NNN active)
- At blocker discovery
- At task completion (move to "idle" until next task)
- On `/handoff` (if claude-kit grows that skill)

## What's NOT in this template

- **Per-task spec details** — those are in `tasks/active/<task>.md`.
  The advertisement just points at the active task by ID.
- **Closing reports** — those are in PR bodies / task-rules.md
  format, not here.
- **AUDIT entries** — those go in `tasks/AUDIT.md`, not here.

The advertisement is **state**, not history. History lives in
`tasks/AUDIT.md`.

## Future direction

When claude-kit grows orchestrator-awareness (per
[`sub-projects.md`](../../sub-projects.md) "Future direction"),
the advertisement protocol will likely extend to include:

- Acknowledgment that orchestrator notices have been read
- Confirmation the sub-kit has applied a convention change
- Cross-references back to orchestrator artifacts the sub-kit
  is aware of

For now, the protocol is one-direction (sub-kit → orchestrator
state advertisement) and read-only on the orchestrator side.
