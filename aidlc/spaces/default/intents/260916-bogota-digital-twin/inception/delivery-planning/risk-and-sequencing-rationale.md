# Risk and Sequencing Rationale: Bogotá Urban Digital Twin

Upstream: `bolt-plan.md` (this stage), `unit-of-work-dependency.md` and `unit-of-work.md` (Units Generation), `team-practices.md` (Practices Discovery: walking skeleton first), `requirements.md` (constraints C-A to C-F, NFR1 to NFR9), `stories.md`, `components.md`, `contract-summary.md`, `mockups.md`, `unit-of-work-story-map.md`. Answers: `delivery-planning-questions.md` (Q1 as resolved by Q7, Q2, Q3, Q6).

A **Bolt** is one build pass over a piece of the work that ends in something that runs. This file explains why the five Bolts in `bolt-plan.md` are in the order they are, where that order departs from the dependency graph, and which risks drove it.

## Heuristic

Two heuristics in sequence, no formal scoring (Q2):

1. **Walking skeleton first** (Cockburn, Crystal Clear): the first Bolt is the thinnest end-to-end slice that touches every architectural layer, so the architecture is proven before features are stacked on it. Affirmed by the team at Practices Discovery, solo and gated.
2. **Risk first** (Boehm, spiral model): the Bolt with the most uncertainty ships next, so the decision it settles is made before dependent work commits to it. The team named the terrain, heights and 3D pipeline as its biggest worry (Q6), and Scope Definition had already sequenced it right after the skeleton.
3. **Value first** for the rest: catalog and ingestion completion (the data guarantees every screen relies on), then the layer panel and wizard (the self-serve success metric), then the 3D view (the last success metric, which depends on both lanes).

Weighted-shortest-job-first was offered and declined: with five Bolts and an affirmed rule for the first two positions, a scoring table would restate the rule in numbers.

## Resolving the first answer

The first answer to "what should we build first" was "the riskiest part first, before any end-to-end slice". That contradicted two recorded decisions: the affirmed walking-skeleton practice in `team-practices.md` (skeleton first, solo and gated, terrain right after), and the dependency graph, where the terrain unit U7 depends on catalog U3 and ingestion U4, which depend on the platform core U1 and identity U2. A follow-up question (Q7) put the conflict to the team, who kept the affirmed practice. The plan therefore reads: skeleton (B1), then terrain (B2). The contradiction and its resolution are recorded in the questions file; no artifact from an earlier stage was changed.

## Deviations from Topological Order

The dependency graph admits several orders. The plan takes one path and departs from a strict unit-by-unit topological order in two places:

| Deviation | What the graph says | What the plan does | Why it is safe |
|---|---|---|---|
| B1 builds slices of U3, U4, U9, U10 before those units are complete | U3 and U4 are large units; U9 and U10 depend on them | B1 builds only the operations the skeleton needs (create layer, list layers, serve tiles, upload one GeoJSON, run the job, record failures, show the layer); B3 and B4 complete the units | A skeleton is by definition a vertical slice; the slice is enumerated in `bolt-plan.md` so completion is traceable, and every table already exists from U1's migrations |
| B2 (U6, U7) before B3 (U3, U4 complete) | U7 depends on U3 and U4 | U7 starts against the B1 slices of the Catalog and Job services | U7 needs create layer, write features and heights, the Job lifecycle and coordinate conversion; all four are in the B1 slice. If U7 needs more, that piece is pulled into B2 (recorded assumption in `bolt-plan.md`) |

B4 running alongside B2 and B3 is not a deviation: `unit-of-work-dependency.md` already allows web units to start against agreed contracts with stubs, and B4 merges only after B3.

## Risk Register

| # | Risk | Likelihood | Impact | Where it is tackled | Mitigation |
|---|---|---|---|---|---|
| R1 | Terrain, heights and 3D pipeline: no prior GIS or 3D experience, large rasters, unfamiliar tooling | High | High | B2 | Sequenced second; bounded to the pilot localidad (C-D); fixtures with known elevation and height so every step is a failing test first; one-week time box for the first terrain tile before committing to tooling; tile set proven in a standalone viewer before B5 |
| R2 | IDECA products unavailable or late (cadastre release; terrain and surface models for the pilot) | Medium | Medium | B2 | Pilot fixture load and terrain pipeline proven on the pilot download first; full-city load deferred with reason if the release is late; see `external-dependency-map.md` |
| R3 | Silent coordinate misplacement (EPSG:9377 to EPSG:4326) | Medium | High | B1 | The single conversion routine ships in the skeleton with its reference-point and round-trip tests; every later loader reuses it (contract C7) |
| R4 | Row policies keyed by the current user unsafe under connection pooling | Medium | High | B1 (mechanism), B3 (negative tests) | Contract C6 fixes the mechanism (set the user inside the request transaction); B3's definition of done requires the negative tests to pass under a pooled connection |
| R5 | Backend-served tiles too slow at city scale | Medium | Medium | B3 | Measured and reported, not thresholded (NFR1); the off-the-shelf tile server can return behind the same URL shape (Contract Design assumption) |
| R6 | 3 to 6 month target with one or two developers | Medium | High | every Bolt | Small Bolts with a definition of done; the skeleton proves deploy-on-merge early; two lanes when two developers are available |
| R7 | Cost ceiling of USD 100 per month exceeded by rasters or tiles | Low | Medium | B2 | Pilot localidad only; cost checked at B2's definition of done before B5 |
| R8 | Cloud account and billing owner not confirmed | Low | Medium | B1 | Not marked as a blocker by the team (Q5); Docker Compose covers local work; B1's deploy step waits for the account without blocking its code |
| R9 | Pilot localidad not chosen when B2 starts | Low | Medium | B2 | Not marked as a blocker by the team (Q5); US9.2 is the first story of B2 and a research-team decision |
| R10 | Skeleton slices grow into full units and B1 stops being thin | Medium | Medium | B1 | The slice table in `bolt-plan.md` is the boundary; anything not in it waits for B3 or B4 |

## Sequencing Summary

| Position | Bolt | Argument |
|---|---|---|
| 1 | B1 Walking skeleton | Affirmed practice; proves the whole architecture (tiles under row policies, job runner, shell and map service, deploy on merge) at the cheapest moment |
| 2 | B2 Pilot localidad data pipeline | Highest uncertainty (R1, R2, R7); settling it early tells the team whether the 3D success metric is reachable before web work depends on it |
| 3 | B3 Catalog and ingestion complete | The data guarantees (visibility enforcement, detection paths, failed rows, upload limit) that every screen relies on; also produces the latency report |
| 4 | B4 Layer panel and add-data wizard | The self-serve success metric; runs alongside B2 and B3 against contracts and merges when the real endpoints exist |
| 5 | B5 3D view | Needs both lanes (tile locations from B2, the panel from B4); the last success metric, so it lands on proven ground |

## Assumptions & Open Questions

- The one-week time box for the first terrain tile in B2 is a planning default the team may change. [assumption]
- If B2 falsifies the 3D hypothesis (tooling or cost), the plan is revisited here before B5 is started; B3 and B4 are unaffected. [assumption]
