# Intent Backlog: Bogotá Urban Digital Twin

Upstream: `intent-statement.md`, `feasibility-assessment.md`, `constraint-register.md`. Answers: `scope-definition-questions.md`. Prioritized with MoSCoW; ordered risk-first as chosen in Q3.

## Prioritized Backlog (proto-Units)

| ID | Proto-Unit | MoSCoW | Order | Depends on | Value | Risk | Source |
|---|---|---|---|---|---|---|---|
| PU-01 | End-to-end slice: one uploaded dataset stored and shown as a map layer | Must | 1 | none | Proves the whole path from file to map | Low | [Q1] [Q7] |
| PU-02 | Pilot-localidad terrain and building heights in 3D | Must | 2 | PU-01 | Meets the 3D success metric; retires the biggest learning risk early | High | [Q2] [Q3] constraint-register C-03, C-04 |
| PU-03 | Official cadastral parcels for Bogotá loaded and inspectable | Must | 3 | PU-01 | Meets the cadastre success metric; base for building heights | Medium | intent-statement |
| PU-04 | Self-serve CSV/JSON upload with coordinate and geometry detection | Must | 4 | PU-01 | Meets the self-serve success metric | Medium | [Q2] |
| PU-05 | Layer panel: toggle, inspect attributes, basic styling | Must | 5 | PU-01 | Makes every layer explorable | Low | [Q2] |
| PU-06 | Geocoding by address and reference codes | Should | 6 | PU-04 | Lets datasets without coordinates be placed on the map | Medium | [Q2] |
| PU-07 | User identity and per-layer visibility, uploader decides | Should | 7 | PU-05 | Admits private datasets | Medium | [Q6] [Q7] |
| PU-08 | Configuration-driven connector for external APIs and web feature services | Could | 8 | PU-04 | Removes manual downloads | Medium | [Q5] |
| PU-09 | Flood-risk model and scenario runs shown as result layers | Could (exploratory) | 9 | PU-02, PU-03 | The digital-twin use case | High | [Q5] intent-statement |

## Dependencies Between Capabilities

- PU-01 is the foundation for everything: storage, map, and the load path exist after it. [Q1]
- PU-02 depends only on PU-01 and is pulled forward deliberately (risk-first) even though the roadmap places it later. [Q3]
- PU-03 and PU-02 together give building heights per parcel; PU-02 can start on terrain alone. constraint-register C-03
- PU-07 gates the admission of private datasets and therefore any private-data requirement in later units. [Q7]

## Value Stream

1. A researcher obtains a dataset (own file, official download, or later an external service). [Q2] [Q5]
2. The dataset is loaded and placed on the map of Bogotá (PU-01, PU-04, PU-06). [Q1] [Q2]
3. The researcher explores it against official cadastre and terrain (PU-03, PU-05, PU-02). intent-statement
4. Later, an analysis produces result layers the researcher explores the same way (PU-09). intent-statement

## Explicitly Excluded

Conversational query interface; orthophoto basemap; private datasets until PU-07 exists; full-city rasters until the pilot pipeline is proven. [Q5] [Q7] constraint-register C-03

## Assumptions & Open Questions

- The order of PU-03, PU-04 and PU-05 after the 3D unit may be re-sequenced at Delivery Planning; the risk-first choice fixes only that PU-02 comes right after PU-01. [assumption]
