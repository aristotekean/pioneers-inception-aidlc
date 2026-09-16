# Feasibility Assessment: Bogotá Urban Digital Twin

Upstream: `intent-statement.md` (Intent Capture). Answers: `feasibility-questions.md`.

## Summary Verdict

The proof of concept is feasible within the confirmed constraints, with two caveats that change the plan rather than block it. First, the confirmed need for per-layer visibility among the four researchers (Q9) means the proof of concept must include user identity and layer permissions from the start, which the initial description had deferred. Second, the raster and 3D work is the highest-uncertainty part of the roadmap for a team with strong web skills but limited GIS and no 3D experience (Q3), so it is treated as a learning-bound risk with a pilot area (Q8) as the mitigation.

Overall rating: **Feasible with managed risk.** The measurable commitment from the intent statement (roadmap phases 1 to 4 in about 3 to 6 months) is achievable with one or two full-time developers (Q4) if the pilot-area strategy holds and the cloud account is confirmed early (Q5).

## Technical Viability

| Area | Assessment | Confidence | Source |
|---|---|---|---|
| Loading external datasets (CSV, JSON, files) and showing them as map layers | Well-understood capability; mature open tooling exists for spatial storage, reprojection and map delivery. Low risk. | High | [Q1] [Q4] |
| Loading official cadastral data from IDECA services | Feasible: IDECA publishes cadastre and terrain products under an open license through downloads and standard web services. The main effort is coordinate-system conversion and the volume of city-wide parcels. | High | [Q1] intent-statement |
| Integrating the partner agency's databases | Feasible but depends on access details not yet known (connection method, schema, refresh cadence). Treated as a dependency. | Medium | [Q1] |
| Address geocoding through a commercial API | Feasible; the free monthly allowance of the leading provider covers a proof-of-concept volume (about 10,000 requests per month as of September 2026). | High | [Q1] [Q6] |
| Terrain and building heights in 3D from LiDAR-derived products | Feasible with the products IDECA lists (terrain and surface models at 50 cm). The pilot-area approach keeps data volume and processing time bounded. Highest learning curve for this team. | Medium | [Q3] [Q8] |
| Per-layer visibility among four researchers | Feasible and small in scope for four known users, but it is new scope relative to the initial description and must be planned into phase 1 or 2. | High | [Q9] |
| Flood-risk simulation (exploratory, not a success metric) | Not assessed for viability here; the intent statement marks it exploratory and outside the measurable commitment. | Not rated | intent-statement |

## Platform Landscape

The team prefers a major public cloud with no account or budget confirmed yet (Q5), a monthly budget under USD 100 (Q6), and a pilot-area strategy for large raster data (Q8). Assessed as of September 2026:

- The preferred provider offers new-account credits plus a permanent free allowance for scale-to-zero container hosting and a small amount of object storage. Its managed relational database has no permanent free tier; the smallest practical instance is roughly USD 7 to 30 per month. This is the only unavoidable recurring cost in the proof of concept. [Q5] [Q6]
- The alternative major provider now offers credits over six months instead of a twelve-month free tier and no free scale-to-zero container hosting, which makes it a worse fit for four intermittent users on this budget. [Q5] [Q6]
- Expected monthly cost with a pilot-area raster set: USD 15 to 40, dominated by the database. A full-city raster set would raise storage and processing cost and should be re-estimated before scaling (see RAID R-03). [Q6] [Q8]
- Data residency: no Colombian rule known to us requires this non-personal research data to stay in-country; recorded as an assumption to confirm with the public-sector partner. [Q2] [assumption]

## Regulatory and Data Scan

- Personal data: none expected in the datasets (Q2), so Colombia's personal-data regime (Ley 1581 de 2012) is not triggered. Should a future dataset carry owner or resident information, a privacy impact assessment becomes mandatory before ingestion. [Q2]
- Open data: cadastral and LiDAR products are published by IDECA under an open license; attribution terms must be respected in any published output. intent-statement
- Private datasets: some datasets are private (Q2) and must be visible only to authorized researchers (Q9). This is a confidentiality control, not a personal-data control, and is the compliance-driven design constraint carried into the constraint register (C-05). [Q2] [Q9]
- Institutional agreements: no pending agreement was reported (Q7). If agency databases (Q1) turn out to require one, it becomes a dependency (RAID D-02). [Q1] [Q7]

## Skills and Capacity

One or two full-time developers (Q4) with strong web skills, limited GIS and no 3D experience, willing to learn (Q3). The vector and web parts of phases 1 and 2 fit this profile. Phases 3 and 4 (terrain, heights, 3D city view) require learning raster processing and 3D delivery; the pilot area (Q8) limits the cost of iteration while learning. A conservative estimate is that phases 3 and 4 consume at least as much calendar time as phases 1 and 2 combined. [Q3] [Q4] [Q8]

## Recommendation

Proceed with the proof of concept under these conditions:

1. Confirm the cloud account and its billing owner before phase 1 starts (Q5); it is the only external setup step and the only recurring cost. [Q5] [Q6]
2. Add user identity and per-layer visibility to the phase 1 or 2 scope rather than deferring access control (Q9). [Q9]
3. Keep the pilot-area strategy for rasters and 3D and re-estimate cost and effort before scaling to the full city (Q8). [Q8]
4. Treat the flood model as exploratory, as the intent statement already does, and do not let it pull effort from phases 1 to 4. intent-statement

## Assumptions & Open Questions

- No Colombian data-residency rule applies to non-personal research data hosted abroad; confirm with the public-sector partner. [assumption]
- The partner agency's databases can be reached over a network connection with read access; the method is not yet known. [assumption]
- The 50 cm terrain and surface products for the pilot localidad are downloadable in a standard raster format without a special request to IDECA. [assumption]
