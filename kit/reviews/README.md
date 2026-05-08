# Reviews

Recurring CTO check-ins. Time-stamped state captures, not feature
work.

The discipline of *consistent shape over time* is what makes trends
visible. Free-form reviews drift; templated reviews compound.

## Cadence

| Review | Cadence | Time | Output |
|---|---|---|---|
| **Weekly** | Friday EOD or Monday AM | ~15 min | `reviews/weekly/YYYY-WW.md` |
| **Monthly** | First week of next month | ~45 min | `reviews/monthly/YYYY-MM.md` |
| **Quarterly** | First week of next quarter | ~half-day | `reviews/quarterly/YYYY-QN.md` |

`YYYY-WW` is ISO week (`date +%G-W%V`). `QN` is `Q1`–`Q4`.

## Templates

- [`_template-weekly.md`](_template-weekly.md) — what shipped, what stuck, what's at risk
- [`_template-monthly.md`](_template-monthly.md) — cost, SLOs, migrations, risks rolled up
- [`_template-quarterly.md`](_template-quarterly.md) — strategic recalibration, principles re-examination, risk-surfacing pass

## Discipline

- **Always done in the same shape.** Filling the template beats
  free-form because consistency makes trends visible across many
  reviews.
- **Honest about misses.** "Migration X stalled" is more useful
  than "Migration X in progress" if it didn't move. The point of
  the review is calibration, not narrative.
- **Action items become artifacts.** Reviews don't end with
  "should do X." They end with: ADR filed, risk surfaced, migration
  opened, roadmap updated, open question added. If a review
  surfaces nothing actionable, that's worth noting too — but
  rarely the case at the macro level.

## AUDIT linkage

Every review writes one line to `AUDIT.md` with the 🎯 emoji
summarizing what was covered + what artifacts came out.

## When NOT to write a review

- Don't backdate. If you skipped a week, note the gap and pick up.
- Don't merge weeks. A week is a week. If two weeks were quiet,
  that's two short reviews — the rhythm matters more than the
  content density.
- Don't replace a review with a chat with the user. The artifact
  is the point. If there's no time for the artifact, write the
  one-line "skipped — reason: …" and move on.

## When NOT to use these templates

- For incident postmortems — see `incidents/`.
- For cross-repo migrations — see `migrations/`.
- For ADRs — see `decisions/`.

Reviews are the *aggregating* artifact. They reference the others;
they don't replace them.
