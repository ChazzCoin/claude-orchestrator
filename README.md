# claude-orchestrator

A Claude Code project shape for **macro architectural truth across multiple
sub-repos**. CTO-assistant scaffolding — not a coding kit, not a task tracker.

## What this is

You have N codebases that should stay coherent (e.g. `api`, `ios`, `web`,
`devops`). Each has its own claude-kit. Each kit thinks locally. Nothing
holds the whole picture.

This is that thing.

The orchestrator is **the macro state** of one company's stack:

- Stack inventory and infra map
- Contracts (data models, endpoints, events)
- Conventions (naming, error handling, auth)
- Platform constraints (runtime / infra realities that constrain design)
- Cross-cutting features and decisions (ADRs)
- **Migrations** — coordinated state transitions across the stack with a
  defined start, blast radius, and close criteria
- A roadmap you maintain, an open-questions parking lot

It does not dispatch work to sub-kits. It does not own a task list. It
holds truth, makes drift visible, and gives you a single reasoning surface
across software, infra, and product.

## Who this is for

You are the CTO (or acting as one) across multiple stacks. You have AI
collaborators inside each repo. You need one place that holds the whole
shape — and a partner who reasons across it with you.

## Use

This repo is the **template**. To stand up an instance for a company:

```sh
gh repo create <company>-orchestrator --private
git clone <company>-orchestrator
cp -R claude-orchestrator/* claude-orchestrator/.claude claude-orchestrator/.gitignore <company>-orchestrator/
cd <company>-orchestrator
# clean: remove this README, replace with your own
```

Then, in Claude Code:

```
/audit
```

The audit skill interviews you and populates the macro state files
(`stack/`, `contracts/`, `conventions/`, etc.). Phase one is straight
macro — the goal is to land truth onto disk where it can be referenced.

## Layout

```
.
├── CLAUDE.md                  # role, conventions, behavior contract
├── stack/                     # services, runtimes, infra topology
├── contracts/                 # data models, endpoints, events (markdown for now)
├── conventions/               # naming, errors, auth — cross-stack rules
├── platform-constraints.md    # runtime constraints that bound design
├── features/                  # cross-cutting feature plans
├── decisions/                 # ADRs (architectural decision records)
├── migrations/                # active/ + closed/ coordinated changes
├── state/                     # regenerated, not hand-edited (manifest, sync-status)
├── roadmap.md                 # active, you maintain
├── open-questions.md          # parking lot for "this looks wrong, deferred"
└── .claude/
    └── skills/                # /audit, /migration, /status, etc.
```

## Sub-kit integration

Sub-kits (claude-kit instances in your `api`, `ios`, `web`, `devops` repos)
read from this orchestrator on demand for context. They never write to it.

In each sub-repo's `CLAUDE.md`, add:

> **Macro architecture and contracts:** see `~/path/to/<company>-orchestrator/`.
> Read `stack/inventory.md`, `contracts/`, and any `features/<x>.md` matching
> current work before making cross-platform decisions.

The orchestrator can write migration notices into each sub-repo's
`.claude/active-migrations.md` so a sub-kit notices an open migration on
session start. That is the only orchestrator → sub-repo write path.

## Conventions

- File names: kebab-case
- Dates: `YYYY-MM-DD`
- Migration / ADR / feature IDs: `YYYY-MM-DD-short-name`
- Markdown is the source format for all macro state
- Generated artifacts live in `state/`; treat them as derived, not authored

## Long-term direction

Markdown contracts are temporary. Once the audit stabilizes and the API
shape is known, migrate `contracts/` to an executable spec (OpenAPI,
protobuf, or shared types package) and generate clients. Drift then
becomes a compile error rather than a discipline.
