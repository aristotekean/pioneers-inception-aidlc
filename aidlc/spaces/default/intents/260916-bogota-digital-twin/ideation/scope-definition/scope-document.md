# Scope Document: Bogotá Urban Digital Twin

Upstream: `intent-statement.md` (Intent Capture), `feasibility-assessment.md` and `constraint-register.md` (Feasibility). Answers: `scope-definition-questions.md`.

## Scope Statement

This piece of work delivers a proof of concept of an urban digital twin of Bogotá for four researchers: a web map on which they can load their own datasets as layers, explore official cadastral data, and see the pilot localidad in 3D with real terrain and building heights. It proves the technical approach before further investment. intent-statement

The product boundary remains the full roadmap with the flood model exploratory (intent-statement), but the **measurable commitment** of this piece of work is the phase 1 to 4 demo. [Q2] intent-statement

## In Scope

| Capability | Priority | Source |
|---|---|---|
| Thin end-to-end slice as the first milestone: one uploaded dataset visible on the map, without login | First demo | [Q1] [Q7] |
| Official cadastral parcels for Bogotá loaded and visible on the map | Must-have (success metric) | intent-statement |
| Self-serve CSV/JSON upload with automatic detection of coordinate columns and native geometry | Must-have | [Q2] |
| Layer panel: toggle layers, click a feature to inspect attributes, basic styling | Must-have | [Q2] |
| Terrain and building heights rendered in 3D for the pilot localidad | Must-have | [Q2] constraint-register C-03 |
| Geocoding by address and by reference codes (UPZ, barrio, manzana) | Should-have | [Q2] |
| User identity and per-layer visibility, with the uploader choosing who can see a private layer | Should-have, after the demo | [Q6] [Q7] |
| Configuration-driven connector for external APIs and web feature services | Could-have (later roadmap phase) | [Q5] |
| Flood-risk model and scenario runs | Could-have, exploratory (later roadmap phase) | [Q5] intent-statement |

## Out of Scope

| Item | Reason | Source |
|---|---|---|
| Conversational query interface over the catalog | Explicitly excluded from this piece of work | [Q5] |
| Orthophoto imagery basemap | Explicitly excluded from this piece of work | [Q5] |
| Private datasets | Stay out of the proof of concept until per-layer visibility exists; the proof of concept works with public data only | [Q7] |
| Full-city rasters | Rasters are limited to the pilot localidad until the pipeline is proven and cost is re-estimated | constraint-register C-03 |
| Virtual reality, edge or sensor computing, generative geometry | Out of scope per the initial description | intent-statement |

## Scope Decisions and Their Effects

- **Private data by exclusion.** Feasibility recorded per-layer visibility as a constraint (C-05) and an open issue (I-01). The scope decision resolves both for this piece of work: the proof of concept ingests public datasets only, so no identity or permission model is required for the demo. The constraint reactivates when private datasets are admitted. [Q7]
- **No login for the first slice.** The first demo slice was first described as sitting behind a login; the identity decision above removes that requirement. [Q1] [Q7]
- **Risk-first sequencing.** The raster and 3D pipeline for the pilot localidad is proven immediately after the first slice, before the upload and layer-panel work, because feasibility rated it the highest learning risk. [Q3] feasibility-assessment
- **No hard deadline.** The 3 to 6 month horizon is a target; milestone demos mark progress. [Q4] intent-statement

## Success Criteria for This Piece of Work

The three success metrics from the intent statement, unchanged: self-serve dataset load visible as a layer; full official cadastre visible and inspectable; 3D terrain and building heights from the LiDAR-derived products, limited to the pilot localidad. intent-statement [Q2]

## Assumptions & Open Questions

- Public datasets alone are enough to demonstrate the three success metrics; if a must-have demo dataset turns out to be private, the identity decision must be revisited. [assumption]
