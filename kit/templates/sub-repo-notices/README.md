# Sub-repo notice templates

Templates for **auto-managed** files the orchestrator writes into
each sub-repo's `.claude/` directory. Each file here corresponds to
a single concern owned by a single skill, regenerated wholesale on
each write, and deleted when its content goes empty.

For **durable, hand-edited** per-repo context (architecture notes,
shared inbox, references), see the sibling
[`sub-repo-shared/`](../sub-repo-shared/) — different discipline,
different file pattern, both directions write. The two patterns
together form the orchestrator's cross-repo write surface.

## Pattern

For each kind of cross-repo notification, the orchestrator drops one
file into the affected sub-repo's `.claude/`:

```
<sub-repo>/.claude/active-<concern>.md
```

The sub-repo's claude-kit reads these files on session start (per
its `task-rules.md` "Active orchestrator notices" section) and
surfaces relevant entries to the user.

**Naming convention:**

- File on disk: `.claude/active-<concern>.md`
- Template here: `kit/templates/sub-repo-notices/<concern>.md.template`
- Skill that owns it: declared in the template's frontmatter or
  body comment.

## Current concerns

| Concern | Template | Owning skill |
|---|---|---|
| Migrations | `migrations.md.template` | `/migration` |

## Adding a new concern (future work)

If the orchestrator should push a new kind of signal into sub-repos
(e.g., active ADRs, pending contract changes, convention shifts),
follow the pattern:

1. **Add a template** here at
   `kit/templates/sub-repo-notices/<concern>.md.template`. Use the
   migrations template as a reference for shape — header with the
   "auto-managed" warning, source-orchestrator pointer, body that
   gets regenerated wholesale on each write.
2. **Add or update the owning skill** to read the template and
   write `<sub-repo>/.claude/active-<concern>.md`.
3. **Update claude-kit's `task-rules.md`** "Active orchestrator
   notices" section to include the new file in the session-start
   read list.
4. **Update this README's table.**

The discipline: one file per concern (not multi-section files),
because each concern has its own owning skill and its own
write-on-empty-delete behavior. Multi-section files would force
multiple skills to coordinate writes, which is the kind of drift
this whole pattern is supposed to prevent.

## What you must NOT do

- **Don't write to a sub-repo without going through these
  templates** (for auto-managed concerns) **or the
  [`sub-repo-shared/`](../sub-repo-shared/) templates** (for durable
  hand-edited content). Ad-hoc writes from skills produce
  inconsistent files and break the sub-kit's read contract.
- **Don't mix the two patterns in one file.** Auto-regenerated
  content overwrites hand edits; hand-edited content drifts from
  auto-managed views of state. One discipline per file.
- **Don't put project-specific content in a template.** Templates
  are generic across all instances of the orchestrator. Anything
  company-specific belongs in the rendered output, not the
  template.
- **Don't accumulate stale entries.** Each write regenerates the
  body wholesale — entries that no longer apply (closed migrations,
  resolved ADRs) drop off automatically. If the body becomes empty,
  delete the file.
