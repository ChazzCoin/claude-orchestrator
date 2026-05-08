# Changelog

## 0.3.0 — 2026-05-08

Wire the sub-project structure. Codifies how the orchestrator
relates to the codebases it tracks — kit-enabled vs non-kit, what's
read, what's written, git discipline, sub-kit advertisement
protocol. Skills are unchanged in this release; the structure is
the foundation skills will use.

**Reference docs (kit-managed, synced via /sync):**

- `kit/sub-projects.md` — the master sub-project document. Two
  flavors (kit-enabled, non-kit), read protocol, four sanctioned
  write paths, sub-kit advertisement protocol, git management
  discipline. Codifies the load-bearing rule: **tasks are for
  coding work; doc/config/spec changes the orchestrator does
  directly via PR.**
- `kit/claude-kit-reference.md` — pinned reference for claude-kit's
  shape (top-level layout, key files, state machine, branch / PR
  conventions). Re-synced manually when claude-kit changes.

**State shape (kit-managed templates + scaffold):**

- `kit/state/sub-repos/_template.md` — per-sub-repo state file
  template. Each registered sub-repo gets
  `state/sub-repos/<name>.md` populated from this. Holds: current
  state (auto), capabilities (auto), affecting-this-repo references
  to all orchestrator artifacts (auto), advertisement (auto), Notes
  (manual, preserved across sync).
- `kit/state/README.md` updated to document the manifest-vs-depth
  split and the auto vs manual fields.
- `kit/templates/sub-repo-advertisement/active.md.template` + README
  — the voluntary sub-kit advertisement protocol shape.

**Manifest format (bootstrap, instance-owned):**

- `bootstrap/state/manifest.md.template` extended with
  `Kit-enabled`, `Last synced HEAD`, `Last synced at`, and
  `State file` (link to per-sub-repo file) fields. The manifest
  becomes a true index pointing into per-sub-repo depth files.

**Operating discipline updates (kit-managed):**

- `kit/orchestrator-rules.md` "Sub-repo write discipline" replaced
  with "Sub-projects — read / write protocol" — references
  sub-projects.md, summarizes the four write paths, summarizes git
  discipline. Old "exactly one place" framing removed (the
  orchestrator can write doc PRs and task specs too, just through
  different paths).
- `bootstrap/CLAUDE.md.template` updated to point at all three
  kit-managed reference docs (orchestrator-rules.md,
  sub-projects.md, claude-kit-reference.md) and summarize the four
  write paths.

**Wiring:**

- `MANIFEST.json` extended with new kit entries (sub-projects.md,
  claude-kit-reference.md, state/sub-repos/_template.md) and
  `state/sub-repos` scaffold dir.
- `bin/init` mirrors the new kit files and scaffolds
  `state/sub-repos/`.

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
