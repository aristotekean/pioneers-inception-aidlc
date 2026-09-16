# Constraint Register: Bogotá Urban Digital Twin

Upstream: `intent-statement.md` (Intent Capture). Answers: `feasibility-questions.md`.

## Technical Constraints

| ID | Constraint | Impact | Source |
|---|---|---|---|
| C-01 | Data enters through three kinds of sources: IDECA open data services, the partner agency's databases, and commercial APIs such as geocoding. | The loading capability must handle files, standard web map services, database connections and rate-limited APIs. | [Q1] |
| C-02 | Official Bogotá datasets use the national projected coordinate system, while web maps expect geographic coordinates. | Every load must convert coordinates; a wrong conversion silently misplaces data. | intent-statement (initial description) |
| C-03 | Terrain and surface models at 50 cm resolution are large; the proof of concept works on one localidad first. | Storage, processing and cost are bounded to the pilot area until the pipeline is proven. | [Q8] |
| C-04 | The team has strong web skills, limited GIS and no 3D experience. | Raster and 3D work carries learning time; estimates for phases 3 and 4 must be conservative. | [Q3] |

## Organizational Constraints

| ID | Constraint | Impact | Source |
|---|---|---|---|
| C-05 | Some datasets are private and must be visible only to authorized researchers, with per-layer visibility among the four users. | User identity and layer permissions are in scope from phase 1 or 2; the initial description's deferral of access control no longer holds. | [Q2] [Q9] |
| C-06 | Capacity is one or two full-time developers. | The roadmap must be sequenced so that one person can deliver phase 1 alone if needed. | [Q4] |
| C-07 | Success is judged on roadmap phases 1 to 4 within about 3 to 6 months; the flood model is exploratory. | Phases 5 to 7 must not consume effort before phases 1 to 4 are demonstrable. | intent-statement |
| C-08 | Decisions are made by the four researchers by consensus, with milestone demos per roadmap phase. | Each phase needs a demo-ready state; scope changes need the group, not one person. | intent-statement |

## Budget and Platform Constraints

| ID | Constraint | Impact | Source |
|---|---|---|---|
| C-09 | Monthly cloud budget under USD 100. | Only scale-to-zero hosting and the smallest managed database tier fit; full-city rasters need a separate cost decision. | [Q6] |
| C-10 | Cloud provider preferred but account and budget not yet confirmed. | Phase 1 cannot be deployed until the account exists; local development is the fallback. | [Q5] |

## Regulatory Constraints

| ID | Constraint | Impact | Source |
|---|---|---|---|
| C-11 | No personal data is expected; Ley 1581 de 2012 is not triggered for the proof of concept. | No privacy impact assessment now; one becomes mandatory if a dataset with personal data is ever proposed. | [Q2] |
| C-12 | Open-license attribution for IDECA products. | Published maps and outputs must carry the required attribution. | intent-statement (initial description) |

## Assumptions & Open Questions

- The private datasets (C-05) are private by institutional policy, not by law; if a legal restriction applies, the control set may need to grow. [assumption]
