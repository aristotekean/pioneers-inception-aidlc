<!-- INVARIANT: examples are single-line HTML comments so a fresh template parses to total=0 (MEMORY_EMPTY). Do NOT un-comment or split across lines. t100 guards this. -->
> This file is kept up to date automatically while the stage runs. Add observations at the review step, not by editing here directly.

## Interpretations
<!-- example: 2026-05-29T10:14:32Z — chose REST over GraphQL; the consuming team only needs CRUD, revisit if subscriptions land -->

- 2026-09-16T20:19:31Z — the user left the style control undecided (Q5); proposed inline expansion as a default and marked it an assumption in all four artifacts rather than blocking the stage on a small layout choice.

## Deviations
- 2026-09-16T20:19:31Z — added four screens the wireframes did not have (sign-in, job detail, visibility dialog, delete confirmation) because login, per-row failure reporting, visibility and deletion became must-haves after the rough mockups.

<!-- example: 2026-05-29T10:14:32Z — skipped the optional caching layer the stage prose suggested; the dataset is small enough that it adds risk -->

## Tradeoffs
<!-- example: 2026-05-29T10:14:32Z — picked TDD over BDD this run; the team is unit-first and the domain is well-understood -->

## Open questions
<!-- example: 2026-05-29T10:14:32Z — confirm the retention window with compliance before the next stage hardens the schema -->
