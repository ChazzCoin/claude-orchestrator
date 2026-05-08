# Changelog

## 0.2.0 — 2026-05-08

Scaffold the CTO operating model. Adds the structural analog of
claude-kit's task-rules + lifecycle for the macro level. Skills for
the new artifact types are stubs — behavior contracts get refined on
first real use.

**Operating discipline (new)**
- `kit/orchestrator-rules.md` — generic CTO operating model: read
  order, artifact map, audit log discipline, migration / risk /
  incident / review lifecycles, sub-repo write rules, decision
  authority, closing-artifact rule. Synced via `/sync` (the
  orchestrator's analog of claude-kit's `task-rules.md`).
- Bootstrap CLAUDE.md.template slimmed — defers operating discipline
  to `orchestrator-rules.md`; keeps role + hard rules + quick
  reference.

**Artifact types (new)**
- `kit/risks/` — risk register pattern with severity × likelihood
  matrix. Instances get `risks/{open,mitigated}/`.
- `kit/incidents/` — cross-stack postmortem pattern + 48h rule +
  action-items-must-become-artifacts discipline.
- `kit/reviews/` — recurring rhythm: weekly / monthly / quarterly
  templates with cadence docs. Instances get
  `reviews/{weekly,monthly,quarterly}/`.

**Skills (stubs)**
- `/risk` — file / update / list / mitigate / accept
- `/incident` — open / update / postmortem / close
- `/review` — weekly / monthly / quarterly with synthesized state pulls
- `/brief` — synthesized CTO orientation across sub-repos +
  orchestrator artifacts

**Bootstrap (instance-owned, new)**
- `tech-vision.md.template` — 12–18mo direction (anchor)
- `tech-principles.md.template` — decision criteria (anchor)
- `vendors.md.template`, `ownership.md.template`, `slos.md.template`,
  `cost-tracking.md.template`, `security-posture.md.template`,
  `compliance.md.template` — operational registries
- `AUDIT.md.template` — chronological macro log

**Wiring**
- `MANIFEST.json` extended with new kit + bootstrap entries and
  `risks/open`, `risks/mitigated`, `reviews/weekly`,
  `reviews/monthly`, `reviews/quarterly` scaffold dirs.
- `bin/init` mirrors the new kit dirs, copies the new bootstrap
  templates, scaffolds the new dirs, and gitignores
  `state/brief-cache.md` for the future `/brief` skill.

## 0.1.0 — 2026-05-07

Initial restructure to claude-kit-style template.

- `kit/` (synced via `/sync`) holds skills + generic ADR / feature /
  migration templates and format docs.
- `bootstrap/` holds one-time `.template` files copied at init —
  CLAUDE.md, README.md, roadmap.md, open-questions.md,
  platform-constraints.md, stack/, contracts/, conventions/,
  state/manifest.md, foundation.json.
- `bin/init` bootstraps a `<company>-orchestrator` instance.
- `MANIFEST.json` declares all file mappings and policies.
- New skills: `/sync` (kit-pull), `/skills` (lister).
- Top-level `CLAUDE.md` removed — match claude-kit. Behavior contract
  lives only at `bootstrap/CLAUDE.md.template` for instances.
- Generated state (`state/sync-status.md`, `state/audit-progress.md`)
  is gitignored in instances.

## 0.1.1 — 2026-05-07

- `kit/templates/sub-repo-notices/migrations.md.template` codifies
  cross-repo write path. Instance gets `.claude/templates/`.
- `/migration` references the template for wholesale-regenerate
  semantics on open and close.
