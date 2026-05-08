# claude-orchestrator

A distributable Claude Code template for **macro architectural truth
across one company's stack**. CTO-assistant scaffolding — not a coding
kit, not a task tracker.

> **Why this exists.** You have N codebases that should stay coherent
> (e.g. `api`, `ios`, `web`, `devops`). Each has its own claude-kit.
> Each kit thinks locally. Nothing holds the whole picture. This is
> that thing.

This repo is the **kit/template**. Run `bin/init` to bootstrap a
per-company instance. `/sync` propagates kit updates into existing
instances.

Modeled on [claude-kit](https://github.com/ChazzCoin/claude-kit) —
same shape (`kit/` synced + `bootstrap/` one-time + `MANIFEST.json` +
`bin/init` + `/sync`), different purpose.

---

## What an instance holds

- Stack inventory and infra map
- Contracts (data models, endpoints, events)
- Conventions (naming, error handling, auth)
- Platform constraints — runtime / infra realities that bound design
- Cross-cutting features and ADRs
- **Migrations** — coordinated state transitions across sub-repos
  with a defined start, blast radius, and close criteria
- A user-maintained roadmap, an open-questions parking lot

The orchestrator does not dispatch work to sub-kits. It does not own
a task list. It holds truth, makes drift visible, and gives the user
one reasoning surface across software, infra, and product.

---

## What's in the kit

```
claude-orchestrator/
├── kit/                          # synced into instances on /sync
│   ├── skills/                   # /audit, /migration, /status, ...
│   ├── decisions/                # ADR template + format docs
│   ├── features/                 # feature plan template + format docs
│   ├── migrations/               # migration template + format docs
│   └── state/README.md
├── bootstrap/                    # one-time, becomes instance property
│   ├── CLAUDE.md.template        # behavior contract w/ {{COMPANY_NAME}}
│   ├── README.md.template
│   ├── roadmap.md.template
│   ├── open-questions.md.template
│   ├── platform-constraints.md.template
│   ├── stack/                    # inventory + infra-map templates
│   ├── contracts/                # models / endpoints / events templates
│   ├── conventions/              # auth / error-handling / naming templates
│   ├── state/manifest.md.template
│   └── foundation.json
├── bin/init                      # bootstrap a <company>-orchestrator
├── MANIFEST.json
├── CHANGELOG.md
└── README.md                     # this file
```

**`kit/`** = synced files. Improvements here propagate to every
instance that runs `/sync`.

**`bootstrap/`** = one-time files. Init copies them once if they
don't exist; they belong to the instance after that and never get
overwritten by `/sync`.

---

## Bootstrapping a new orchestrator instance

```sh
gh repo create <company>-orchestrator --private
git clone https://github.com/<you>/<company>-orchestrator
git clone https://github.com/ChazzCoin/claude-orchestrator /tmp/claude-orchestrator
/tmp/claude-orchestrator/bin/init <company>-orchestrator
cd <company>-orchestrator
# fill in the {{COMPANY_NAME}} / {{COMPANY_PURPOSE}} placeholders in
# CLAUDE.md and README.md, commit, push.
```

The init script:

1. Mirrors `kit/skills/` into `<target>/.claude/skills/`.
2. Mirrors `kit/decisions/`, `kit/features/`, `kit/migrations/*.md`,
   `kit/state/README.md` into the matching instance directories.
3. Copies `bootstrap/*.template` into the instance — only if the
   target file doesn't already exist.
4. Stamps `<target>/.claude/foundation.json` with the kit's commit
   SHA so `/sync` knows what to compare against.
5. Scaffolds `migrations/active/` and `migrations/closed/` with
   `.gitkeep`.
6. Drops a default `.gitignore` if missing.
7. Prints next steps.

---

## Skills shipped

Run `/skills` inside an instance after init to see the live list.
Current set:

| Skill | Purpose |
|---|---|
| `/audit` | Interview-driven macro audit — stack, contracts, conventions, constraints. v1 entry point. |
| `/decision` | Draft a macro-level ADR. |
| `/feature` | Author a cross-cutting feature plan. |
| `/migration` | Open, update, or close a cross-repo migration. |
| `/onboard` | Synthesize current orchestrator state for a new collaborator. |
| `/register` | Register a new sub-repo in `state/manifest.md`. |
| `/skills` | List every locally-defined skill. |
| `/status` | Current macro picture at a glance. |
| `/sync` | Pull updates from this kit into the instance. |
| `/sync-check` | Surface drift between sub-repos and the orchestrator's contracts. |

---

## Iterating

### "I improved a skill in instance A — get it into the kit"

1. In instance A, edit `.claude/skills/<name>/SKILL.md`.
2. Copy that change into the claude-orchestrator repo's
   `kit/skills/<name>/SKILL.md`.
3. Commit + push to claude-orchestrator.
4. In instance B (and every other instance), run `/sync` and accept
   the skill update.

`/sync` is the one-way valve. It reads from the kit, never writes
back. Pushing improvements upstream is a manual git operation —
intentional, so changes are reviewed.

### "I want to override a kit file in one instance only"

Edit the file locally. `/sync` detects the divergence as a "local
override" and won't overwrite it without approval. The override
gets recorded in `.claude/foundation.json`.

### "The kit shipped a bad change — I want to roll back"

```sh
# In claude-orchestrator:
git revert <bad-commit>
git push
# Then in each instance:
/sync   # picks up the revert
```

Or pin to an older commit by editing `.claude/foundation.json`'s
`pinned_sha` field manually.

---

## Design principles

- **Generic where possible, specific where it matters.** Skills and
  ADR/feature/migration templates are generic; instance-specific
  content lives in `bootstrap/` files that the instance owns
  post-init.
- **One-way sync.** Kit → instance. Improvements flow back via
  manual git, not auto-push. Reviews matter.
- **Never auto-commit.** Skills that modify files leave changes
  staged or unstaged for human review.
- **Honest about drift.** `/sync` surfaces conflicts, doesn't
  silently merge.

---

## Long-term direction

Markdown contracts are temporary. Once a company's audit stabilizes
and the API shape is known, migrate `contracts/` to an executable
spec (OpenAPI, protobuf, or a shared types package) and generate
clients. Drift then becomes a compile error rather than a discipline.

## License

MIT. Copy, fork, hack on it. If you make it better, send a PR.
