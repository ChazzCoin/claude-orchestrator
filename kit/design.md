# Design standards

How the orchestrator tracks the company's visual identity and
collateral specs, and how design discipline gets validated across
sub-projects.

Generic across all instances — synced via `/sync` (kit-managed).
Instance-specific design content lives in the `design/` directory
(bootstrap, instance-owned).

---

## What lives where

The orchestrator is **the registry**, not the design tool. Canonical
high-fidelity assets live in design tools (Figma, Sketch, Adobe
suite) and storage (Drive, Notion, S3). The orchestrator holds:

- **Tokens** (color hex codes, typography scale, spacing, motion) —
  source of truth in markdown. Other places copy from here.
- **Brand identity** (palette intent, voice, tone) — markdown.
- **Asset registry** — pointers to canonical files in design tools
  / storage, with current version and owner.
- **Collateral specs** — descriptions of business cards, signatures,
  letterhead, swag, presentation templates.
- **Per-platform packages** — mobile graphics, web design system
  references, email templates.

If a high-fidelity asset is small (logo SVG/PNG at canonical sizes,
favicon, app icon at common resolutions), it MAY be committed to
the orchestrator under `design/assets/` for git-diffable reference.
Big binaries stay in design tools.

---

## Source of truth discipline

For each kind of design knowledge, exactly one place is canonical:

| Kind | Canonical | How to update |
|---|---|---|
| Color tokens (hex codes) | `design/brand.md` (this orchestrator) | Direct edit + ADR for any change |
| Typography scale + families | `design/brand.md` | Same |
| Spacing / sizing scale | `design/brand.md` | Same |
| Voice + tone guidelines | `design/brand.md` | Direct edit |
| Logo source files | Whatever's in `design/assets.md` (Figma, Drive, etc.) | Update there, then update `assets.md` to point at the new version |
| Component / pattern designs | Design tool (Figma, etc.) | Update there |
| Mobile graphics | Design tool, with index in `design/mobile.md` | Update tool, update index |
| Email signatures | `design/materials.md` (markdown spec) + asset link | Direct edit |
| Business cards | `design/materials.md` (markdown spec) + asset link | Direct edit |

The orchestrator's job is to be the **authoritative pointer** —
when a sub-project asks "what hex is our primary?" the answer
comes from `design/brand.md`, not from a stale Figma frame
someone copied a year ago.

---

## When updates happen

Design changes are CTO-level state changes. Cadence:

- **Token / brand changes** are paired with an ADR (`/decision`)
  because they propagate to every sub-project. The ADR explains
  why; the `brand.md` update is the durable spec.
- **Cross-repo rebrand** opens a migration (`/migration`) — the
  per-repo state tracks how each sub-repo adopted the new tokens.
- **Asset version bumps** (logo refresh, etc.) — direct edit to
  `design/assets.md` with a date stamp. AUDIT entry with 🎨.
- **New collateral type** (new business card variant, new email
  template) — direct edit to `design/materials.md`.
- **Per-platform package updates** — direct edit to `design/<platform>.md`.

For coding sub-projects: changes to design TOKENS in a sub-repo's
code are still **code changes** (per the Code boundary in
[`sub-projects.md`](sub-projects.md)). The orchestrator updates
`design/brand.md` directly; the corresponding code change in each
sub-repo goes through a task or migration.

---

## Validation across the system

When a PR touches design / UI in a sub-project, the orchestrator's
PR review runs a **Design fit** check (see
[`pr-reviews/_template.md`](pr-reviews/_template.md)):

| Check | Reads from |
|---|---|
| Are tokens used (vs hardcoded values)? | `design/brand.md` (token list) |
| Do colors match `design/brand.md` palette? | `design/brand.md` |
| Does typography match the type scale? | `design/brand.md` |
| Do spacing values match the scale? | `design/brand.md` |
| Do new assets follow naming + folder conventions? | `design/assets.md` |
| Per-platform conformance | `design/<platform>.md` |

When materials are produced (new business card design, signature
update, marketing collateral), the validation is:

- Does it reference current canonical tokens? (Often: pull hex
  codes fresh from `design/brand.md`, not from a 3-month-old slide.)
- Is it added to the asset registry?
- Is the source file in the right design tool / storage?

---

## Cross-platform drift

Brand drift across sub-projects (iOS using `#0066CC`, web using
`#0077CC`, both supposed to be "primary blue") is the failure
mode this discipline prevents. Surface drift via:

- Quarterly review (`/review quarterly` includes a design
  consistency check)
- PR review's Design fit check on any UI-touching PR
- `/audit` design phase if formally re-auditing

Drift surfaced becomes:

- A risk in `risks/open/` if material
- A migration in `migrations/active/` if cross-repo correction is
  needed
- An ADR in `decisions/` if the tokens themselves were the
  problem

---

## What this discipline is NOT

- **Not a Figma replacement.** Design tools are the right place
  for high-fidelity work. The orchestrator holds the
  authoritative-text version (tokens, voice, asset pointers).
- **Not a brand guidelines document.** Brand guidelines are a
  deliverable produced FROM `design/` (or live alongside it).
  This is the working source-of-truth.
- **Not a marketing CMS.** Marketing copy variants for campaigns
  live wherever marketing keeps them. Design tracks the visual
  language.

---

## Updating this discipline

If the company's design practice grows beyond what this structure
holds (e.g., motion design becomes its own track, accessibility
specs need their own file, internationalization adds locale-
specific design), extend the `design/` directory. Update this
file with the new structure. ADR for the structural change.
