# Team Allocation: Bogotá Urban Digital Twin

Upstream: `bolt-plan.md` (this stage), `unit-of-work.md` and `unit-of-work-dependency.md` (Units Generation), `team-practices.md` (Practices Discovery), `requirements.md` (constraint C-A: one or two developers), `components.md` (Domain Design), `stories.md`, `mockups.md`, `unit-of-work-story-map.md`, `contract-summary.md`. Answers: `delivery-planning-questions.md` (Q4).

A **Bolt** is one build pass over a piece of the work that ends in something that runs. A **mob** is the group that builds a Bolt together; here it is one AI developer with one or two human developers reviewing. Team Formation was skipped by this workflow's scope, so there is no team roster to reference: every Bolt is executed by the AI developer (`aidlc-developer-agent`) in the build repositories, with the human developers approving each Bolt and running the self-review checklist before merge. With a single mob, no Program Board (the cross-team dependency board used when several teams share a plan) is needed; the two lanes below are branches, not teams.

## Bolt-to-Mob Assignment

| Bolt | Name | Units | Mob | Lane | Human role |
|---|---|---|---|---|---|
| B1 | Walking skeleton | U1, U2, U8; slices of U3, U4, U9, U10 | AI developer | solo | one developer approves the skeleton explicitly before B2 and B4 start |
| B2 | Pilot localidad data pipeline | U6, U7 | AI developer | backend (jobs) | developer A: chooses the pilot localidad with the researchers, downloads the IDECA products, reviews spatial tests |
| B3 | Catalog and ingestion complete | U3, U4, U5 | AI developer | backend (API) | developer A: reviews row-policy tests and the latency report |
| B4 | Layer panel and add-data wizard | U9, U10 | AI developer | web | developer B (or A when alone): reviews against `mockups.md`, runs the researcher walkthrough |
| B5 | 3D view | U11 | AI developer | web | developer B (or A): reviews on a tablet |

## Lanes and Concurrency

Answer Q4: two Bolts at a time when independent, one backend and one web, each on its own branch.

- **Backend lane**: B2, then B3. Both touch the backend codebase (the job runner shares the ingestion module's Job contract), so they run one after the other.
- **Web lane**: B4 starts after B1 against the agreed contracts (C2, C4) with stubs where B3 has not merged yet, and merges after B3; B5 follows B4 and B2.
- **One developer**: the lanes serialize in merge order (B2, B3, B4, B5). Nothing in the plan changes except elapsed time.
- **Two developers**: developer A owns the backend lane, developer B the web lane; the contracts in `contract-summary.md` are the handshake between them (providers deploy first, consumers ignore unknown fields).

## Skills and Gaps

| Skill | Needed in | Status | Mitigation |
|---|---|---|---|
| FastAPI, SQLAlchemy, Alembic, PostGIS | B1, B2, B3 | present | none needed |
| Angular strict mode, the map engine's 2D API | B1, B4 | present | none needed |
| Raster processing: terrain-model to terrain tiles, surface minus terrain per parcel | B2 | gap (no prior GIS or 3D experience) | B2 sequenced right after B1; fixtures with known elevations and heights (US9.1) so every step is a failing test first; time-box the first terrain tile at one week before choosing tooling |
| 3D tile set generation and the map engine's 3D API | B2, B5 | gap | prove the tile set in a standalone viewer at B2 before any web work depends on it |
| Database row policies under connection pooling | B1 (slice), B3 | partial | contract C6 fixes the mechanism (set the user inside the request transaction); negative tests in B3 |
| GitHub Actions, Cloud Run deploy on merge | B1 | present | first deploy is part of B1's definition of done |

## Communication Norms

- One shared environment: the lane that merges announces it, since every merge deploys.
- Contract changes are additive only; a provider change is merged before its consumer (Contract Ownership Rules in `contract-summary.md`).
- Per-row failures and secrets rules are checked in every Bolt's self-review, not only where they were first built.
- Escalation: a Bolt that cannot meet its definition of done inside its time box goes back to this plan for a split or a re-order, with the reason recorded in `risk-and-sequencing-rationale.md`.

## Assumptions & Open Questions

- The two human developers are interchangeable across lanes; the A and B labels above are a convenience, not a fixed assignment. [assumption]
- The researchers (Andrés, Camila, Laura and the fourth account) are demo participants and data owners, not builders. [assumption]
