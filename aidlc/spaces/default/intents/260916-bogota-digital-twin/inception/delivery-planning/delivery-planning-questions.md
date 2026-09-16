# Delivery Planning Questions

This stage orders the work into Bolts: a Bolt is one build pass over a piece of the work that ends in something that runs. Context carried forward: 11 units of work with an acyclic dependency graph; the team affirmed building a thin end-to-end slice first (a walking skeleton: a minimal version that runs the whole way through, built first to prove the pieces connect); Scope Definition chose risk-first sequencing with the terrain and 3D pipeline right after that slice; one or two developers full-time; one shared environment deployed on every merge; separate repositories per deployable. Two corrections from Contract Design are carried into the plan: U5 tile-serving is now documentation owned by U3 (tiles are served by the backend), and U11 web-3d reaches tiles through U10's map service.

## Q1. What should we build first?

A. The thin end-to-end slice first (sign-in, one uploaded dataset visible on the map), then the riskiest part (terrain and 3D for the pilot localidad), then the rest by value
B. The thin slice first, then everything by value (cadastre, upload, layer panel) with terrain and 3D last
C. The riskiest part first, before any end-to-end slice
D. Not yet defined
X. Other (please specify)

[Answer]: C. The riskiest part first, before any end-to-end slice

## Q2. Should the work be ranked with a formal scoring model?

A formal model such as weighted-shortest-job-first scores each piece by value, urgency and risk reduction against its size, and ships the highest score first.

A. No formal model; order by the affirmed rule (thin slice first, then risk, then value) and explain each Bolt's position
B. Yes, score every Bolt with value, urgency and risk against size, with risk weighted highest
C. Not yet defined
X. Other (please specify)

[Answer]: A. No formal model; order by the affirmed rule (thin slice first, then risk, then value) and explain each Bolt's position

## Q3. How big should one Bolt be?

A. One unit of work per Bolt (about 11 Bolts)
B. Several related units bundled per Bolt (for example backend core with identity; layer panel with tiles), about 6 to 7 Bolts
C. Thin slices that cut across units (a feature per Bolt regardless of unit boundaries)
D. Not yet defined
X. Other (please specify)

[Answer]: B. Several related units bundled per Bolt (for example backend core with identity; layer panel with tiles), about 6 to 7 Bolts

## Q4. Can several Bolts be built at the same time?

A. One Bolt at a time, given one or two developers and one shared environment
B. Two at a time when independent (one backend, one web), each on its own branch
C. Not yet defined
X. Other (please specify)

[Answer]: B. Two at a time when independent (one backend, one web), each on its own branch

## Q5. What outside the team could hold up the build? (select all that apply)

For each selected item, the plan records the owner, lead time, the Bolt it blocks and the fallback.

A. The cloud account and billing owner (not yet confirmed)
B. Downloading the official cadastre release from IDECA
C. Downloading the terrain and surface products for the pilot localidad from IDECA
D. Choosing the pilot localidad (a research-team decision)
E. None of these holds anything up
X. Other (please specify)

[Answer]: B, C. Downloading the official cadastre release from IDECA; downloading the terrain and surface products for the pilot localidad from IDECA

## Q6. What worries you most about this build, so it is tackled early?

A. The terrain, heights and 3D pipeline: new skills, large data
B. Getting per-layer visibility right in tiles and attributes
C. Upload robustness: geometry detection, coordinate conversion, failed rows
D. The 3 to 6 month target with one or two developers
E. Not yet defined
X. Other (please specify)

[Answer]: A. The terrain, heights and 3D pipeline: new skills, large data

## Q7. Follow-up: your Q1 answer conflicts with the affirmed team practice and with the unit dependencies

You chose "the riskiest part first, before any end-to-end slice" (Q1, option C). Two recorded decisions point the other way: the team affirmed at Practices Discovery that the walking skeleton (a thin sign-in, upload, see-it-on-the-map slice) runs first, solo and gated, with the terrain and 3D pipeline immediately after it (`team.md`, Walking Skeleton; requirements constraint C-E). And the terrain and 3D unit (U7) depends on the catalog (U3) and ingestion (U4) units, which depend on the platform core (U1) and identity (U2), so nothing 3D can run before most of the backend exists anyway. At build time the affirmed practice also overrides a plan that puts the skeleton anywhere but first. Which order governs?

A. Keep the affirmed practice: walking skeleton first (Bolt 1), terrain and 3D pipeline second (Bolt 2), then the rest by value
B. Riskiest first as answered: build the backend core, identity, catalog, ingestion and the terrain pipeline with no web interface, then the thin slice; this also requires changing the affirmed Walking Skeleton practice before Construction
C. Walking skeleton first, but fold a bounded terrain spike into Bolt 1 (load one terrain tile for the pilot localidad end to end) so the riskiest technology is touched on day one
X. Other (please specify)

[Answer]: A. Keep the affirmed practice: walking skeleton first (Bolt 1), terrain and 3D pipeline second (Bolt 2), then the rest by value

## Consolidated Summary Confirmation

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
