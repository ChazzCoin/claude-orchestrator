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

**Anchors** *(rarely change)*

- `tech-vision.md` — long-arc direction (12–18 months)
- `tech-principles.md` — how decisions get made
- `platform-constraints.md` — runtime / infra realities

**Macro state** *(populated via `/audit`, evolved over time)*

- `stack/` — inventory + infra map
- `contracts/` — data models, endpoints, events
- `conventions/` — naming, error handling, auth

**Operational registries** *(filled as data exists)*

- `vendors.md` — third-party deps with renewals + risk profile
- `ownership.md` — who owns what, on-call, escalation
- `slos.md` — service-level objectives per sub-repo
- `cost-tracking.md` — cloud + vendor spend trend
- `security-posture.md` — audits, gaps, practices
- `compliance.md` — frameworks (SOC2 / GDPR / etc.)

**Active state** *(changes regularly)*

- `roadmap.md` — what's being steered toward
- `open-questions.md` — parking lot
- `state/manifest.md` — registered sub-repos
- `AUDIT.md` — chronological log of macro actions

**Artifacts** *(one per concern, with templates synced from kit)*

- `decisions/` — ADRs
- `features/` — cross-cutting feature plans
- `migrations/{active,closed}/` — coordinated state transitions
- `risks/{open,mitigated}/` — risk register
- `incidents/` — cross-stack postmortems
- `reviews/{weekly,monthly,quarterly}/` — recurring CTO check-ins

The orchestrator does not dispatch work to sub-kits. It does not own
a task list. It holds truth, makes drift visible, and gives the user
one reasoning surface across software, infra, and product.

---

## Cross-repo write path

Sub-kits **read** the orchestrator on demand. The orchestrator
**writes** into sub-repos in exactly one place:
`<sub-repo>/.claude/active-<concern>.md` — files the orchestrator
drops so a sub-kit notices coordination signals on session start.

Today only `active-migrations.md` exists; the structure is in place
to add more (active ADRs, pending contract changes, etc.) without
re-architecting. The pattern is documented in
[`kit/templates/sub-repo-notices/README.md`](kit/templates/sub-repo-notices/README.md).

For the read-side to actually work, the sub-repo's claude-kit needs
the matching session-start rule. See `claude-kit`'s `task-rules.md`
"Active orchestrator notices" section.

---

## What's in the kit

```
claude-orchestrator/
├── kit/                              # synced into instances on /sync
│   ├── orchestrator-rules.md         # CTO operating discipline (analog of task-rules.md)
│   ├── skills/                       # /audit, /migration, /risk, /incident, /review, ...
│   ├── decisions/                    # ADR template + format docs
│   ├── features/                     # feature plan template + format docs
│   ├── migrations/                   # migration template + format docs
│   ├── risks/                        # risk register template + format docs + matrix
│   ├── incidents/                    # incident template + format docs + 48h rule
│   ├── reviews/                      # weekly/monthly/quarterly templates + cadence docs
│   ├── state/README.md
│   └── templates/                    # files skills render into sub-repos
│       └── sub-repo-notices/         #   migrations.md.template, ...
├── bootstrap/                        # one-time, becomes instance property
│   ├── CLAUDE.md.template            # company-specific behavior + {{COMPANY_NAME}}
│   ├── README.md.template
│   ├── roadmap.md.template
│   ├── open-questions.md.template
│   ├── platform-constraints.md.template
│   ├── tech-vision.md.template       # 12–18mo direction
│   ├── tech-principles.md.template   # decision criteria
│   ├── vendors.md.template
│   ├── ownership.md.template
│   ├── slos.md.template
│   ├── cost-tracking.md.template
│   ├── security-posture.md.template
│   ├── compliance.md.template
│   ├── AUDIT.md.template             # chronological macro log
│   ├── stack/                        # inventory + infra-map templates
│   ├── contracts/                    # models / endpoints / events templates
│   ├── conventions/                  # auth / error-handling / naming templates
│   ├── state/manifest.md.template
│   └── foundation.json
├── bin/init                          # bootstrap a <company>-orchestrator
├── MANIFEST.json
├── CHANGELOG.md
└── README.md                         # this file
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

1. Copies `kit/orchestrator-rules.md` to `<target>/.claude/`.
2. Mirrors `kit/skills/` into `<target>/.claude/skills/`.
3. Mirrors `kit/templates/` into `<target>/.claude/templates/`
   (used by skills that render into sub-repos).
4. Mirrors `kit/decisions/`, `kit/features/`, `kit/migrations/`,
   `kit/risks/`, `kit/incidents/`, `kit/reviews/`,
   `kit/state/README.md` into the matching instance directories.
5. Copies `bootstrap/*.template` into the instance — skip-if-exists.
   That includes CLAUDE.md, README, roadmap, open-questions,
   platform-constraints, tech-vision, tech-principles, vendors,
   ownership, slos, cost-tracking, security-posture, compliance,
   AUDIT, plus stack/, contracts/, conventions/, state/manifest.md.
6. Stamps `<target>/.claude/foundation.json` with the kit's commit
   SHA so `/sync` knows what to compare against.
7. Scaffolds `migrations/{active,closed}/`, `risks/{open,mitigated}/`,
   `reviews/{weekly,monthly,quarterly}/` with `.gitkeep`.
8. Drops a default `.gitignore` if missing.
9. Prints next steps.

---

## Skills shipped

Run `/skills` inside an instance after init to see the live list.
Current set:

| Skill | Purpose |
|---|---|
| `/audit` | Interview-driven macro audit — stack, contracts, conventions, constraints. v1 entry point. |
| `/brief` | Synthesized daily orientation across sub-repos + orchestrator artifacts (stub). |
| `/decision` | Draft a macro-level ADR, referencing `tech-principles.md`. |
| `/feature` | Author a cross-cutting feature plan. |
| `/incident` | Log a cross-stack incident, draft postmortem (stub). |
| `/migration` | Open, update, list, or close a cross-repo migration. |
| `/onboard` | Synthesize current orchestrator state for a new collaborator. |
| `/register` | Register a new sub-repo in `state/manifest.md`. |
| `/review` | Run weekly / monthly / quarterly review (stub). |
| `/risk` | File a risk in the register, update, or mitigate (stub). |
| `/skills` | List every locally-defined skill. |
| `/status` | Current macro picture at a glance. |
| `/sync` | Pull updates from this kit into the instance. |
| `/sync-check` | Surface drift between sub-repos and the orchestrator's contracts. |

**(stub)** = skill scaffolding is in place; behavior contract is
provisional and gets refined on first real use.

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
