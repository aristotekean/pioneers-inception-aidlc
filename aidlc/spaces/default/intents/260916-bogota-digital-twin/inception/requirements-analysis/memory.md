<!-- INVARIANT: examples are single-line HTML comments so a fresh template parses to total=0 (MEMORY_EMPTY). Do NOT un-comment or split across lines. t100 guards this. -->
> This file is kept up to date automatically while the stage runs. Add observations at the review step, not by editing here directly.

## Interpretations
<!-- example: 2026-05-29T10:14:32Z — chose REST over GraphQL; the consuming team only needs CRUD, revisit if subscriptions land -->

- 2026-09-16T19:49:27Z — 'best effort' map responsiveness (Q5) is not testable as a threshold; recorded NFR1 as a measure-and-report requirement (median and p95 tile latency under 4 users) so the inception guardrail on measurable requirements still holds.

## Deviations
- 2026-09-16T19:49:27Z — carried the practices-interview auth reversal into FR1 as must-haves and re-admitted per-layer visibility (FR4.6) although scope-definition had excluded it; the newer confirmed answer wins and the scope artifact stays untouched.

<!-- example: 2026-05-29T10:14:32Z — skipped the optional caching layer the stage prose suggested; the dataset is small enough that it adds risk -->

## Tradeoffs
<!-- example: 2026-05-29T10:14:32Z — picked TDD over BDD this run; the team is unit-first and the domain is well-understood -->

## Open questions
<!-- example: 2026-05-29T10:14:32Z — confirm the retention window with compliance before the next stage hardens the schema -->
