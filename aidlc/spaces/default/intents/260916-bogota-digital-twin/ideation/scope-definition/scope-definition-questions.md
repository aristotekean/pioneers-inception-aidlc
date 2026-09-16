# Scope Definition Questions

Context carried forward: the product boundary is the full roadmap with the flood model exploratory; success is judged on roadmap phases 1 to 4 (cadastral layer on the map, self-serve upload and layer panel, terrain, building heights and the 3D city view) in about 3 to 6 months; per-layer visibility among the four researchers is a confirmed constraint; rasters start on a pilot localidad; one or two developers full-time. These are not re-asked.

## Q1. What is the smallest slice that would count as the first demonstrable milestone?

Your document's phase 1 is "schema, manual load of official cadastral parcels, map showing the layer". The first demo sets the rhythm for the milestone demos you chose.

A. Phase 1 as written: official cadastral parcels visible on the map, loaded manually
B. Phases 1 and 2 together: cadastre on the map plus self-serve CSV/JSON upload and the layer panel
C. A thin end-to-end slice first: one uploaded dataset visible on the map behind a login, before any official data
D. Not yet defined
X. Other (please specify)

[Answer]: C. A thin end-to-end slice first: one uploaded dataset visible on the map behind a login, before any official data

## Q2. Which capabilities are must-haves for the phase 1 to 4 demo? (select all that apply)

Anything not selected becomes a should-have or could-have in the backlog, not an exclusion.

A. Self-serve CSV/JSON upload with automatic detection of coordinate columns and native geometry
B. Layer panel: toggle layers, click a feature to inspect its attributes, basic styling
C. User identity and per-layer visibility among the four researchers
D. Terrain and building heights rendered in 3D for the pilot localidad
E. Geocoding by address and by reference codes (UPZ, barrio, manzana)
X. Other (please specify)

[Answer]: A, B, D

## Q3. How should the work be sequenced?

Your document lists phases in dependency order. Feasibility rated the raster and 3D pipeline as the highest learning risk for this team.

A. Roadmap order as written: dependency-first, phases 1 to 4 in sequence
B. Risk-first: prove the raster and 3D pipeline on the pilot area right after phase 1, before the upload and layer panel work
C. Value-first: self-serve upload and layer panel before completing the official cadastre load
D. Walking skeleton first: a thin end-to-end slice with identity and one layer, then roadmap order
E. Not yet defined
X. Other (please specify)

[Answer]: B. Risk-first: prove the raster and 3D pipeline on the pilot area right after phase 1, before the upload and layer panel work

## Q4. Are there hard deadlines tied to specific capabilities?

You reported no known blockers and a 3 to 6 month target; this confirms whether that target is a deadline.

A. None: 3 to 6 months is a target, not a deadline
B. A fixed date for the first milestone demo (please specify via Other)
C. A fixed date tied to the 3D city view or a report to the funding body
D. Not yet defined
X. Other (please specify)

[Answer]: A. None: 3 to 6 months is a target, not a deadline

## Q5. What is explicitly out of this piece of work, even though the roadmap mentions it? (select all that apply)

Naming what is out prevents scope creep during construction. Items not selected stay in the backlog as later phases.

A. The flood-risk model and scenario runs (exploratory; may move to a later piece of work)
B. The conversational query interface over the catalog
C. The configuration-driven connector for external APIs and web feature services
D. The orthophoto imagery basemap
E. Nothing else is out; everything in the roadmap stays in the backlog
X. Other (please specify)

[Answer]: B, D

## Q6. Who decides which researchers can see a private layer?

Per-layer visibility needs an owner for the decision at upload time.

A. The researcher who uploads the dataset chooses who can see it
B. One designated administrator among the four assigns visibility
C. Visibility follows the dataset's source institution automatically
D. Not yet defined
X. Other (please specify)

[Answer]: A. The researcher who uploads the dataset chooses who can see it

## Q7. Follow-up: is user identity with per-layer visibility a must-have or not?

Your answers pull in two directions. You did not select "user identity and per-layer visibility" as a must-have (Q2), yet the first demo slice sits "behind a login" (Q1), the uploader decides who sees a private layer (Q6), and feasibility recorded per-layer visibility as a constraint (C-05) that supersedes the document's deferral of access control. The backlog needs one answer.

A. Must-have: identity and per-layer visibility are in the phase 1 to 4 demo, as the feasibility constraint says
B. Split: login is a must-have for the demo, per-layer visibility is a should-have delivered right after it
C. Neither is needed for the demo: private layers stay out of the proof of concept until visibility exists
D. Not yet defined
X. Other (please specify)

[Answer]: C. Neither is needed for the demo: private layers stay out of the proof of concept until visibility exists

## Consolidated Summary Confirmation

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
