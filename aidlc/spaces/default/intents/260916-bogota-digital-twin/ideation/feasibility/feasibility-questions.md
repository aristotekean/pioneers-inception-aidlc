# Feasibility Questions

Context carried from Intent Capture: proof of concept, four researchers from a university, a private company and a public-sector agency, full roadmap boundary with the flood model exploratory, success judged on roadmap phases 1 to 4, first demo targeted at 3 to 6 months. Your architecture document already states the team stack (PostgreSQL, Python, Go, Angular) and a GCP preference, so those are not re-asked; the questions below cover what it leaves open.

## Q1. Which existing systems must the proof of concept integrate with?

Your document names IDECA and Catastro Distrital as data sources exposed through downloads, OGC services (WMS/WFS) and ESRI REST, and Google Geocoding for addresses. Integration constraints drive the connector work and the risk register.

A. Only IDECA open data: file downloads plus their OGC/ESRI REST services
B. IDECA plus internal databases of the partner public-sector agency
C. IDECA plus commercial APIs (for example Google Geocoding)
D. All of the above: IDECA, agency databases and commercial APIs
E. None beyond manual file uploads for the proof of concept
X. Other (please specify)

[Answer]: D. All of the above: IDECA, agency databases and commercial APIs (user's words: "I would say ideca, agency and commercial apis")

## Q2. Do any datasets carry personal data or institutional restrictions?

Cadastral and LiDAR products are published under an open license, but cadastral records linked to owners, or agency datasets, may fall under Colombian personal-data law (Ley 1581 de 2012, habeas data) or data-sharing agreements. This decides whether access control and data handling rules are a constraint now rather than later.

A. Only open public data; no personal data is expected in the proof of concept
B. Some datasets may include personal data (owners, addresses tied to people), so Ley 1581 applies
C. Agency datasets come with institutional restrictions or a data-sharing agreement
D. Both personal data and institutional restrictions are expected
E. Not yet assessed
X. Other (please specify)

[Answer]: X. Other — the data won't have personal data but can be public or private

## Q3. What is the team's skill profile for GIS, raster processing and 3D visualization?

Your document lists "team size and GIS experience available for the raster and 3D pipelines" as an open question. The raster (COG, terrain tiles) and 3D (3D Tiles, CesiumJS) pipelines are the least common skills in a typical web team.

A. Strong GIS (PostGIS, GDAL, QGIS) but no 3D or CesiumJS experience
B. Strong web (Angular, Python) but limited GIS and no 3D experience
C. Both GIS and 3D experience available in the team
D. Neither GIS nor 3D experience; the team will learn as it builds
E. Not yet defined
X. Other (please specify)

[Answer]: B. Strong web (Angular, Python) but limited GIS and no 3D experience (user's words: "i would say b but i can learn")

## Q4. How much development capacity is available for the proof of concept?

The 3 to 6 month target for phases 1 to 4 is only realistic against a known capacity.

A. One developer part-time
B. One or two developers full-time
C. Three or more developers
D. The researchers build it themselves alongside their research (no dedicated developers)
E. Not yet defined
X. Other (please specify)

[Answer]: B. One or two developers full-time

## Q5. Which cloud platform will host the proof of concept, and is it approved?

Your document prefers GCP but leaves the provider undecided. Hosting also decides where large rasters live and whether managed PostGIS is available.

A. GCP, with an account and budget approved
B. GCP preferred, but no account or budget confirmed yet (credits or free tier expected)
C. AWS instead of GCP
D. University or agency servers, or local-only development for the proof of concept
E. Not yet defined
X. Other (please specify)

[Answer]: B. GCP preferred, but no account or budget confirmed yet (credits or free tier expected) — chosen after the user asked for a GCP vs AWS free-tier and tooling comparison

## Q6. What is the monthly cloud budget order of magnitude?

Full-city terrain and surface rasters plus managed PostGIS have a recurring cost; the budget bounds the storage and compute choices.

A. Under USD 100 per month
B. USD 100 to 500 per month
C. USD 500 to 2,000 per month
D. Grant-funded credits with no fixed monthly cap
E. Not yet defined
X. Other (please specify)

[Answer]: A. Under USD 100 per month

## Q7. Are there organizational blockers or dependencies outside the team's control? (select all that apply)

A. A data-sharing or partnership agreement with IDECA or the agency is still pending
B. Cloud account, procurement or institutional approval is still pending
C. Researcher availability is limited by competing priorities
D. A fixed academic or grant deadline constrains the timeline
E. None known
X. Other (please specify)

[Answer]: E. None known (user's words: "none")

## Q8. Should the raster and 3D pipelines start with the full city or a pilot area?

The 50 cm urban terrain and surface models for all of Bogotá are large (tens to hundreds of gigabytes before conversion). Your document suggests developing on one localidad first; confirming this bounds the storage, processing time and cost for the proof of concept.

A. A single localidad or pilot area first, then scale to the full city if the pipeline works
B. The full city from the start, accepting the processing and storage cost
C. Vector data for the full city, but rasters only for a pilot area
D. Not yet defined
X. Other (please specify)

[Answer]: A. A single localidad or pilot area first, then scale to the full city if the pipeline works

## Q9. Follow-up: what does "private" data require from the proof of concept?

You said datasets carry no personal data but can be public or private (Q2), while your document states access control is not required initially for four internal users. These only fit together if "private" means something specific.

A. Only that the tool itself is not publicly reachable; all four researchers may see every layer
B. Per-layer visibility restrictions among the four researchers (some layers hidden from some users)
C. Private datasets must never leave the owning institution, so they stay on that institution's infrastructure
D. Not yet defined
X. Other (please specify)

[Answer]: B. Per-layer visibility restrictions among the four researchers (some layers hidden from some users)

## Consolidated Summary Confirmation

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
