# PR reviews — orchestrator-grade

The orchestrator's PR review is **macro-level**: does this PR fit
the contract? Different from claude-kit's per-repo `/review` (which
catches line-level issues — security, performance, correctness
within the repo's codebase).

The two reviews compose. Sub-kit catches "is this line right";
orchestrator catches "is this consistent with everything else
we've decided."

## When to invoke an orchestrator review

Not every PR. Only when one of:

- **The PR touches a sub-repo affected by an active migration** —
  cross-check that the diff matches the migration's per-repo plan.
- **The PR touches a contract surface** — data models, endpoints,
  events. The orchestrator's `contracts/*.md` should change too if
  the PR is changing the contract; if not, that's drift.
- **The PR introduces, removes, or swaps a vendor.**
- **The PR changes auth, schemas, or anything in a
  `conventions/*.md` domain.**
- **The PR was authored by the orchestrator itself** (`chore/orch-*`
  branch). Self-review by the same agent is a circle — surface to
  the user; have a real human reviewer in the loop.
- **The user explicitly asks for an orchestrator review.**

For routine per-repo coding work that doesn't touch any of the
above, the sub-kit's `/review` is sufficient. The orchestrator
only steps in when macro context is needed.

## What gets checked

The checklist pulls from orchestrator artifacts. See
[`_template.md`](_template.md) for the structured review form.

| Check | Reads from |
|---|---|
| Task / spec match | `<sub>/tasks/active/TASK-NNN.md`, or PR body if no spec |
| Code does what the spec says | the diff itself |
| Code is clean, organized, well-structured | the diff |
| Naming convention | `conventions/naming.md` |
| Error handling | `conventions/error-handling.md` |
| Auth | `conventions/auth.md` |
| Contract surface honored | `contracts/{models,endpoints,events}.md` |
| Platform constraints honored | `platform-constraints.md` |
| Tech principles honored or consciously broken | `tech-principles.md` |
| Cross-platform fit (does this need a related change in another repo?) | `state/manifest.md` + per-sub-repo state |
| Active migration alignment | `migrations/active/<id>.md` per-repo plan |
| Surfaces a new risk? | (none — file via `/risk` if so) |
| Repeats a recent failure mode? | `incidents/`, recent postmortems |
| Small gotchas | the diff + general experience |

The "cross-platform fit" check is what makes this an orchestrator
review and not just a per-repo one. Example: an API endpoint
change that requires an iOS or web client update, but the PR
touches only the API. Per-repo review wouldn't catch it; macro
review does.

## What a review produces

Three possible outputs, not mutually exclusive:

1. **Approval** — `✅ approved`. Review filed; PR comment posted
   (with user approval) summarizing the check; AUDIT entry with
   `🔍 Reviewed <repo>#<N> — approved`.
2. **Conditional** — `⚠ conditional`. Specific concerns listed,
   posted as PR comment, PR may merge once concerns are addressed.
   AUDIT entry with `🔍 Reviewed <repo>#<N> — conditional`.
3. **Request changes** — `❌ request changes`. Blockers listed,
   posted as PR comment with explicit "macro-level concerns"
   framing. PR's author addresses; orchestrator does NOT fix the
   PR (that's the sub-kit / author's job). AUDIT with
   `🔍 Reviewed <repo>#<N> — changes requested`.

**Spawn artifacts** when applicable. A review that surfaces a real
risk, convention gap, contract drift, or missing ADR files the
artifact in addition to commenting:

- New risk → `/risk` to file in `risks/open/`.
- Convention gap → ADR via `/decision` to update `conventions/*.md`.
- Contract drift → migration via `/migration` if cross-cutting, or
  ADR if a one-off correction.
- Repeated failure → reference the existing incident postmortem; if
  the failure mode is novel, file an incident.

## Storage

Reviews are filed at `pr-reviews/YYYY-MM-DD-<repo>-pr<N>.md` (one
per review). Frontmatter tracks the PR URL, sub-repo, status,
related migrations / ADRs / risks. Body has the structured
checklist. See [`_template.md`](_template.md).

If the same PR is re-reviewed (PR updates after initial review),
either:
- Append a new "Re-review" section to the existing file, OR
- File a new `YYYY-MM-DD-<repo>-pr<N>-rereview.md`

The first is preferred for short re-reviews; the second when the
PR has changed materially.

## What you must NOT do

- **Don't duplicate per-repo line-level review.** That's claude-
  kit's `/review`. The orchestrator review is the layer above.
- **Don't auto-approve or auto-merge.** Reviews surface; the user
  decides what to do with them.
- **Don't post a comment to the PR without showing the user
  first.** The user reviews the review.
- **Don't review your own PRs alone.** A `chore/orch-*` PR was
  authored by the orchestrator; reviewing it from the same agent
  is a circle. Either escalate to the user (manual review) or
  defer until claude-kit grows orchestrator-awareness for second
  opinion.
- **Don't fix the PR.** When changes are requested, the orchestrator
  documents what needs to change. The author (sub-kit or user)
  fixes it.

## Relationship to other artifacts

- **Sub-kit `/review`** — runs first, catches line-level issues.
  Result lives on the PR (comments).
- **Orchestrator review** (this) — runs second, catches macro fit.
  Result lives in `pr-reviews/` + posted comment + spawned
  artifacts.
- **Migration validation checklist** — when a PR is part of an
  active migration, the orchestrator review feeds into the
  migration's per-repo state (⚪ → 🟡 → ✅). The PR review approves
  the per-repo state move, not the migration close.
- **Incidents / risks** — reviews can surface new risks or
  reference existing incidents to flag repeated patterns.

## When NOT to write an orchestrator review

- **Pure refactor inside one sub-repo with no cross-cutting
  concern** — the sub-kit's review suffices.
- **Documentation-only PR** — unless the doc is a contract / ADR /
  conventions file, sub-kit owns it.
- **Test-only PR** — sub-kit owns it.
- **Dependency bumps that don't change behavior** — sub-kit owns
  the audit; macro review only if the dependency change has
  cross-repo or compliance implications.

## What "done" looks like

- A file in `pr-reviews/<id>.md` with status filled in.
- A draft PR comment shown to the user; posted (via `gh pr comment`)
  on user approval; or skipped if the user prefers to write it
  themselves.
- AUDIT entry with 🔍.
- Any spawned artifacts (risks, ADRs) filed.
