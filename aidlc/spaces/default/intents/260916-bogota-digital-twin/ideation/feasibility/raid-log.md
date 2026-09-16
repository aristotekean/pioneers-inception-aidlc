# RAID Log: Bogotá Urban Digital Twin

Upstream: `intent-statement.md` (Intent Capture). Answers: `feasibility-questions.md`. Likelihood and impact are rated Low, Medium or High.

## Risks

| ID | Risk | Likelihood | Impact | Treatment | Owner | Source |
|---|---|---|---|---|---|---|
| R-01 | Raster and 3D pipelines take longer than planned because the team is learning GIS and 3D while building. | High | High | Mitigate: pilot area first, time-box a learning spike before phase 3, keep phases 1 and 2 independent of 3D. | Research team | [Q3] [Q8] |
| R-02 | Per-layer visibility was not in the original plan and adds scope to phases 1 and 2. | High | Medium | Mitigate: keep the permission model minimal (four named users, per-layer allow list); plan it into phase 1 or 2 explicitly. | Research team | [Q9] |
| R-03 | Scaling rasters from the pilot area to the full city exceeds the USD 100 monthly budget. | Medium | Medium | Mitigate: re-estimate storage and processing cost at the end of phase 3 before any full-city run; accept a pilot-only 3D view for the proof of concept if needed. | Research team | [Q6] [Q8] |
| R-04 | Cloud account and billing owner are not confirmed, delaying the first deployment. | Medium | Medium | Mitigate: confirm the account before phase 1; develop locally in the meantime. | Research team | [Q5] |
| R-05 | Coordinate conversion errors misplace official data on the map. | Medium | High | Mitigate: verify the coordinate system of every download and validate a known landmark after each load. | Developers | intent-statement (initial description) |
| R-06 | With one or two developers, an absence stalls the whole roadmap. | Medium | Medium | Accept for the proof of concept; keep documentation current so a second person can pick up. | Research team | [Q4] |

## Assumptions

| ID | Assumption | Validation | Owner | Source |
|---|---|---|---|---|
| A-01 | No Colombian data-residency rule applies to non-personal research data hosted abroad. | Confirm with the public-sector partner before phase 1 deployment. | Public-sector researcher | [Q2] [assumption] |
| A-02 | Pilot-area terrain and surface products are downloadable in a standard raster format without a special request. | Download one tile during phase 1. | Developers | [Q8] [assumption] |
| A-03 | The partner agency's databases are reachable with read access over a network connection. | Confirm connection method and credentials before phase 6. | Agency researcher | [Q1] [assumption] |

## Issues

| ID | Issue | Status | Source |
|---|---|---|---|
| I-01 | The initial description states access control is not required initially, but per-layer visibility is now confirmed. The description's statement is superseded by the confirmed answer. | Open until the requirements stage records the permission model. | [Q9] intent-statement |

## Dependencies

| ID | Dependency | Type | Needed by | Source |
|---|---|---|---|---|
| D-01 | IDECA open data services and downloads (cadastre, terrain, surface models). | External data provider | Phase 1 | [Q1] |
| D-02 | Partner agency database access. | Institutional | Phase 6 | [Q1] |
| D-03 | Commercial geocoding API account and key. | External service | Phase 5 | [Q1] |
| D-04 | Cloud account with billing owner. | Platform | Phase 1 deployment | [Q5] |

## Assumptions & Open Questions

None beyond the Assumptions table above.
