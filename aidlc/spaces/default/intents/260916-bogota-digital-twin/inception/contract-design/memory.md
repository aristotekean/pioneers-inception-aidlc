<!-- INVARIANT: examples are single-line HTML comments so a fresh template parses to total=0 (MEMORY_EMPTY). Do NOT un-comment or split across lines. t100 guards this. -->
> This file is kept up to date automatically while the stage runs. Add observations at the review step, not by editing here directly.

## Interpretations
<!-- example: 2026-05-29T10:14:32Z — chose REST over GraphQL; the consuming team only needs CRUD, revisit if subscriptions land -->

- 2026-09-16T21:27:42Z — 'use the simplest way for the mvp' on tile identity was read as the backend serving tiles itself inside the request transaction (row policies apply, no exposed tile server); stated the interpretation in the summary the user confirmed, and recorded the off-the-shelf tile server as a later swap behind the same URL shape.

## Deviations
<!-- example: 2026-05-29T10:14:32Z — skipped the optional caching layer the stage prose suggested; the dataset is small enough that it adds risk -->

## Tradeoffs
- 2026-09-16T21:27:42Z — separate repositories (Q6) with additive-only contracts (Q7) over one repository: the user's choice; recorded the coordination rule (providers deploy first, consumers ignore unknown fields) so the same-commit assumption in Q3 is replaced rather than left dangling.

<!-- example: 2026-05-29T10:14:32Z — picked TDD over BDD this run; the team is unit-first and the domain is well-understood -->

## Open questions
<!-- example: 2026-05-29T10:14:32Z — confirm the retention window with compliance before the next stage hardens the schema -->
