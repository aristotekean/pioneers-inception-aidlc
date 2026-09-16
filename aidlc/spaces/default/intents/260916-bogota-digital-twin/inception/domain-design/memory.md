<!-- INVARIANT: examples are single-line HTML comments so a fresh template parses to total=0 (MEMORY_EMPTY). Do NOT un-comment or split across lines. t100 guards this. -->
> This file is kept up to date automatically while the stage runs. Add observations at the review step, not by editing here directly.

## Interpretations
<!-- example: 2026-05-29T10:14:32Z — chose REST over GraphQL; the consuming team only needs CRUD, revisit if subscriptions land -->

- 2026-09-16T20:46:15Z — Q1 (four blocks) and Q5 (separate Terrain3D) together yield five backend blocks; recorded five and said so at the summary rather than forcing the Q1 count.

## Deviations
<!-- example: 2026-05-29T10:14:32Z — skipped the optional caching layer the stage prose suggested; the dataset is small enough that it adds risk -->

## Tradeoffs
- 2026-09-16T20:46:15Z — row policies (Q2) over a tile proxy: one enforcement point, but the off-the-shelf tile server must run each request as the user; flagged the mechanism as a Contract Design decision in ADR-002 instead of picking one here.

<!-- example: 2026-05-29T10:14:32Z — picked TDD over BDD this run; the team is unit-first and the domain is well-understood -->

## Open questions
- 2026-09-16T20:46:15Z — confirm at Contract Design how the tile server passes the current user to the database (validated per-request parameter vs per-user connection); the choice affects whether the tile server stays 'no custom code'.

<!-- example: 2026-05-29T10:14:32Z — confirm the retention window with compliance before the next stage hardens the schema -->
