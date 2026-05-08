---
type: quarterly
period: YYYY-QN          # Q1 | Q2 | Q3 | Q4
filed: YYYY-MM-DD
---

# Quarterly review — {{period}}

## Headline

<3–5 sentences. The quarter's macro shape. What changed in our
position vs. where we wanted to be 3 months ago?>

## Roadmap reality check

<Read `roadmap.md` "Now / Next / Later" sections. For each entry
that was active or queued at the start of the quarter:>

| Roadmap item | Status at start | Status now | Reason for change |
|---|---|---|---|
| _(populate)_ | | | |

**Roadmap re-shape:** _(if any items moved Now → Later, Later →
dropped, etc., explain the bend)_

## Tech principles re-examination

<Read `tech-principles.md`. For each principle:>

- **{{Principle}}** — Still right? Did anything this quarter test
  it? Should it be reworded, dropped, replaced?

If a principle changed, file an ADR superseding the old version.

## Tech vision check-in

<Read `tech-vision.md`. Is the 12–18-month direction still right?
What's the gap between "today" and "12 months out"? Did this
quarter narrow it, widen it, or rotate it?>

## Risk-surfacing pass *(mandatory — quarterly)*

The quarterly review is the prompt to look for risks we haven't
filed yet. Don't skip this. Empty `risks/open/` is suspicious.

For each sub-repo + each cross-cutting concern (auth, data,
infra, vendors, compliance), ask: what's the worst realistic
failure mode in the next 12 months? Anything new since last
quarter?

- _(file new risks via /risk; reference IDs here)_

## Reliability + cost roll-up

<From the last 3 monthly reviews:>

- **SLO trends** — improving / degrading / flat per sub-repo
- **Cost trend** — quarterly total + YoY comparison
- **Incident count** — by severity, this quarter vs last quarter

## Vendor + compliance review

- **Vendors with renewals in next quarter:** _(from `vendors.md`)_
- **Compliance milestones in next quarter:** _(from
  `compliance.md`)_
- **Security posture changes this quarter:** _(from
  `security-posture.md`)_

## Big bet for next quarter

<One bet. What's the one shift this quarter is going to test? File
as a roadmap "Now" item if not already there.>

## Open questions resolved / parked / aging

<From `open-questions.md`:>

- **Resolved this quarter:** _(IDs)_
- **Newly parked:** _(IDs)_
- **Aging > 6 months:** _(IDs — these need a decision or an
  explicit "we've decided not to decide")_

## What I want to bend next quarter

1. _(populate)_
2. _(populate)_
3. _(populate)_

## Notes

_(unstructured — anything worth keeping for the year-end review)_
