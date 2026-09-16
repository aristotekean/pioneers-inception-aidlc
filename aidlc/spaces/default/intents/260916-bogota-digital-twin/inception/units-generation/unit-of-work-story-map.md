# Story Map to Units: Bogotá Urban Digital Twin

Upstream: `stories.md`, `unit-of-work.md`, `components.md`, `requirements.md`, `decisions.md`. Each story is mapped to its implementing unit; stories that cross units list the owning unit first.

## Story Assignment

| Story | Unit ID | Directory | Also touches | Order within unit |
|---|---|---|---|---|
| US1.1 | U2 | u2-identity | U8 (screen) | 1 |
| US1.2 | U2 | u2-identity | U8 (client session) | 2 |
| US1.3 | U2 | u2-identity | — | 3 |
| US1.4 | U2 | u2-identity | U1 (secret scan in CI) | 4 |
| US2.1 | U4 | u4-ingestion | U9 (wizard step 1) | 1 |
| US2.2 | U4 | u4-ingestion | U9 (step 2) | 2 |
| US2.3 | U4 | u4-ingestion | U9 (step 2) | 3 |
| US2.4 | U4 | u4-ingestion | U9 (step 2) | 4 |
| US2.5 | U4 | u4-ingestion | U9 (steps 1 and 2) | 5 |
| US2.6 | U4 | u4-ingestion | U9 (step 3, status) | 6 |
| US2.7 | U4 | u4-ingestion | U9 (job detail) | 7 |
| US2.8 | U4 | u4-ingestion | U9 (job detail) | 8 |
| US2.9 | U4 | u4-ingestion | — | 9 |
| US2.10 | U10 | u10-web-layers | U3 (extent) | 1 |
| US2.11 | U9 | u9-web-upload | — | 1 |
| US3.1 | U6 | u6-jobs-cadastre | U4 (conversion), U3 | 1 |
| US3.2 | U10 | u10-web-layers | U3 | 2 |
| US3.3 | U6 | u6-jobs-cadastre | — | 2 |
| US3.4 | U6 | u6-jobs-cadastre | — | 3 |
| US4.1 | U10 | u10-web-layers | — | 3 |
| US4.2 | U10 | u10-web-layers | U3 (styles) | 4 |
| US4.3 | U10 | u10-web-layers | U3 (attributes) | 5 |
| US4.4 | U8 | u8-web-shell | U4 (job count) | 1 |
| US4.5 | U3 | u3-catalog | U10 (dialog) | 1 |
| US4.6 | U3 | u3-catalog | U10 (dialog), U9 (step 3) | 2 |
| US4.7 | U10 | u10-web-layers | — | 6 |
| US4.8 | U8 | u8-web-shell | — | 2 |
| US4.9 | U3 | u3-catalog | U5 (tiles) | 3 |
| US5.1 | U11 | u11-web-3d | U8 (toggle control) | 1 |
| US5.2 | U7 | u7-jobs-terrain3d | U11 (view) | 1 |
| US5.3 | U11 | u11-web-3d | U7 (boundary) | 2 |
| US5.4 | U11 | u11-web-3d | — | 3 |
| US5.5 | U11 | u11-web-3d | — | 4 |
| US5.6 | U7 | u7-jobs-terrain3d | U3 (heightM) | 2 |
| US5.7 | U7 | u7-jobs-terrain3d | U11 (view) | 3 |
| US6.1 | U1 | u1-platform-core | — | 1 |
| US6.2 | U1 | u1-platform-core | — | 2 |
| US6.3 | U5 | u5-tile-serving | — | 1 |
| US7.1 | U8 | u8-web-shell | U9, U10, U11 (their labels) | 3 |
| US7.2 | U8 | u8-web-shell | U10, U11 (touch) | 4 |
| US8.1 | deferred | — | later piece of work | — |
| US8.2 | deferred | — | later piece of work | — |
| US8.3 | deferred | — | later piece of work | — |
| US8.4 | deferred | — | later piece of work (Analysis unit) | — |
| US9.1 | U1 | u1-platform-core | U4, U7 (fixture consumers) | 3 |
| US9.2 | U7 | u7-jobs-terrain3d | — | 0 (first) |

## Cross-cutting Stories

- US7.1 (Spanish interface) is owned by U8 through the resource strings and applies to every web unit.
- US7.2 (tablet) is owned by U8 through the drawer and touch targets; U10 and U11 add their touch gestures.
- US4.6 and US4.9 are owned by U3 (data and policies) with their dialogs in U10 and the step-3 control in U9.
- US2.6 and US2.7 are owned by U4 with their screens in U9.

## Coverage Verification

| Check | Result |
|---|---|
| Every story assigned | 42 of 46 assigned to a unit; 4 later-capability stories (US8.1 to US8.4) deferred by scope |
| Every unit has stories | U1: 3; U2: 4; U3: 3; U4: 9; U5: 1; U6: 3; U7: 4; U8: 4; U9: 1; U10: 6; U11: 4 |
| Stories per unit sum | 42 |

## Assumptions & Open Questions

- The order within each unit is a suggestion for the unit's own work; Delivery Planning owns the order across units. [assumption]
