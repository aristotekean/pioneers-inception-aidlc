# Units Generation Questions

Context carried forward: six building blocks (Identity, Catalog, Ingestion, Terrain3D, Analysis, WebApp) with an acyclic dependency graph; your document proposes a modular monolith backend, a separate job runner, a frontend, and off-the-shelf tile servers; the affirmed practices are trunk-based with one shared environment and one or two developers. These questions decide how blocks are grouped into units of work and how units may depend on each other. Build order is decided at Delivery Planning, not here.

## Q1. How should the building blocks be grouped into units of work?

A. By deployable: one backend unit (Identity, Catalog, Ingestion services and the shared schema), one jobs unit (Ingestion job runner and Terrain3D pipelines), one web unit (WebApp), one tile-serving unit (tile server configuration and row policies as a spec), and Analysis as a thin later unit
B. One unit per building block (six units)
C. By user-facing slice: walking-skeleton slice, sign-in, upload, layer panel, cadastre, 3D, each cutting across blocks
D. Not yet defined
X. Other (please specify)

[Answer]: A. By deployable

## Q2. How coarse should the units be?

A. Coarse: 3 to 5 units, each a deployable or a contract
B. Medium: 6 to 8 units, splitting the largest deployables by capability
C. Fine: 9 or more units, close to one per story group
D. Not yet defined
X. Other (please specify)

[Answer]: C. Fine: 9 or more units, close to one per story group

## Q3. What is the deployment model?

A. Modular monolith: one backend API process, one job runner process, one static frontend, off-the-shelf tile servers, one database (your document's proposal)
B. One independently deployed service per building block
C. Not yet defined
X. Other (please specify)

[Answer]: A. Modular monolith: one backend API process, one job runner process, one static frontend, off-the-shelf tile servers, one database

## Q4. May independent units be built in parallel?

A. Yes: units with no dependency between them may proceed in parallel (for example the web unit against an agreed contract while the backend is built)
B. No: strictly one unit at a time in dependency order
C. Not yet defined
X. Other (please specify)

[Answer]: A. Yes: units with no dependency between them may proceed in parallel

## Q5. How do units integrate?

A. A JSON HTTP contract between the web unit and the backend; database schema owned by the backend unit and applied by migrations; tiles read by the tile server under row policies; jobs share the backend's database and schema
B. Events between backend and jobs (queue), HTTP for the rest
C. Not yet defined
X. Other (please specify)

[Answer]: A. A JSON HTTP contract between the web unit and the backend; database schema owned by the backend unit and applied by migrations; tiles read by the tile server under row policies; jobs share the backend's database and schema

## Q6. Follow-up: "by deployable" gives 4 or 5 units, but you asked for 9 or more. Which shape do you want?

Grouping by deployable (Q1) yields about five units; fine granularity (Q2) asks for nine or more. Both can hold only if the deployables are split by capability.

A. Split the deployables by capability: the backend into core schema and platform, identity, catalog, ingestion; jobs into cadastre load and terrain/3D; the web into shell and sign-in, upload wizard, layer panel, 3D view; plus the tile-serving contract: about 11 units that still deploy as one backend, one job runner and one frontend
B. Keep 4 or 5 coarse deployable units and drop the fine-granularity answer
C. Not yet defined
X. Other (please specify)

[Answer]: A. Split the deployables by capability: about 11 units that still deploy as one backend, one job runner and one frontend

## Consolidated Summary Confirmation

- Grouping: by deployable, split by capability into about 11 units (backend core and platform, identity, catalog, ingestion; jobs for the cadastre load and for terrain and 3D; web shell with sign-in, upload wizard, layer panel, 3D view; tile-serving contract; thin Analysis unit later)
- Deployment: modular monolith, one backend API process, one job runner, one static frontend, off-the-shelf tile servers, one database
- Parallel work allowed between independent units
- Integration: JSON HTTP contract between web and backend; schema owned by the backend and applied by migrations; tiles under row policies; jobs share the backend database

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
