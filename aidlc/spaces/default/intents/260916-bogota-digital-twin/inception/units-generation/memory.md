<!-- INVARIANT: examples are single-line HTML comments so a fresh template parses to total=0 (MEMORY_EMPTY). Do NOT un-comment or split across lines. t100 guards this. -->
> This file is kept up to date automatically while the stage runs. Add observations at the review step, not by editing here directly.

## Interpretations
<!-- example: 2026-05-29T10:14:32Z — chose REST over GraphQL; the consuming team only needs CRUD, revisit if subscriptions land -->

- 2026-09-16T21:16:14Z — 'by deployable' (Q1) and 'fine, 9+' (Q2) conflicted; a follow-up resolved it as deployables split by capability (11 units that deploy as three processes), recorded as such.

## Deviations
- 2026-09-16T21:16:14Z — the decomposition plan section could not live inside the questions file after the summary confirmation (the receipt refused an extra H2); presented the plan in chat with a logged decision/answer pair instead and kept the plan content in unit-of-work.md.

<!-- example: 2026-05-29T10:14:32Z — skipped the optional caching layer the stage prose suggested; the dataset is small enough that it adds risk -->

## Tradeoffs
- 2026-09-16T21:16:14Z — made platform-core (U1) own every table's migration so later units add columns and policies rather than tables; simpler for a two-developer team than per-unit schema ownership, at the cost of one bigger first unit.

<!-- example: 2026-05-29T10:14:32Z — picked TDD over BDD this run; the team is unit-first and the domain is well-understood -->

## Open questions
<!-- example: 2026-05-29T10:14:32Z — confirm the retention window with compliance before the next stage hardens the schema -->
