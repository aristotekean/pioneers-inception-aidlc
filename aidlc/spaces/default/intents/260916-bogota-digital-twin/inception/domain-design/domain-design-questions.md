# Domain Design Questions

Context carried forward: your architecture document proposes a modular monolith with `catalog`, `ingestion` and `models` modules, a map engine behind an adapter in the frontend, a tile server that reads the database directly, and rasters in object storage. Since then, per-user login and per-layer visibility became must-haves (stories US1, US4.6, US4.9), the cadastre and height computations became developer-run jobs (US3.1, US5.6), and the affirmed practices require module boundaries through service layers. These questions settle the building blocks; deployment topology is decided at Units Generation.

## Q1. Which backend building blocks do you want?

A. Four: Identity (users, sessions), Catalog (datasources, layers, features, visibility), Ingestion (upload jobs, detection, coordinate conversion, per-row failures), Analysis (scenarios; later phases)
B. Three: fold Identity into Catalog, keep Ingestion and Analysis
C. Five: as A, plus a separate Terrain3D block for terrain, building heights and 3D tiles
D. Not yet defined
X. Other (please specify)

[Answer]: A. Four: Identity, Catalog, Ingestion, Analysis

## Q2. How is per-layer visibility enforced when tiles and attributes are served?

Your document has the tile server reading the database directly with no custom code. Visibility must hold for direct tile requests (story US4.9), so one of two mechanisms is needed.

A. A dedicated TileAccess component sits in front of the tile server: it checks the session and the layer's allowed users, then proxies the tile
B. Database-level row policies keyed by the current user, so the tile server enforces visibility without a proxy
C. Not yet defined
X. Other (please specify)

[Answer]: B. Database-level row policies keyed by the current user, so the tile server enforces visibility without a proxy

## Q3. How is the frontend structured?

A. A Map Shell with feature areas (sign-in, layer panel, upload wizard, feature popup, view mode, job detail) that talk to the map only through a map adapter
B. One application component with the map engine used directly from each feature
C. Not yet defined
X. Other (please specify)

[Answer]: B. One application component with the map engine used directly from each feature

## Q4. Who owns the background job record (status, timings, failed rows)?

A. Ingestion owns one Job entity used by uploads, the cadastre load and the height computation
B. Each kind of job owns its own status record
C. Not yet defined
X. Other (please specify)

[Answer]: A. Ingestion owns one Job entity used by uploads, the cadastre load and the height computation

## Q5. Who owns terrain, building heights and 3D tile generation?

A. A separate Terrain3D block owns raster processing, the per-parcel height computation and 3D tile generation, storing heights on the parcel through Catalog
B. Ingestion owns them as more job types
C. Not yet defined
X. Other (please specify)

[Answer]: A. A separate Terrain3D block owns raster processing, the per-parcel height computation and 3D tile generation, storing heights on the parcel through Catalog

## Q6. Follow-up: your document keeps a map adapter "regardless" of the engine; Q3 drops it. Which do you intend?

Your architecture document (principle 4) says the Angular frontend reaches the map only through a `MapAdapter` so the engine can change without rewriting application logic. Your Q3 answer uses the engine directly from each feature. Both are legitimate for a proof of concept; I need the one you mean.

A. Drop the adapter for the proof of concept: features use the map engine directly; this supersedes principle 4 of the document
B. Keep the adapter but as one application shell: a single map service wraps the engine, and features call that service, not the engine
C. Not yet defined
X. Other (please specify)

[Answer]: B. Keep the adapter but as one application shell: a single map service wraps the engine, and features call that service, not the engine

## Consolidated Summary Confirmation

- Backend blocks: Identity, Catalog, Ingestion, Analysis (Q1) plus a separate Terrain3D block (Q5): five in total
- Visibility enforcement for tiles and attributes: database-level row policies keyed by the current user, no proxy
- Frontend: one application shell with a single map service wrapping the engine; features call the service, not the engine
- Jobs: Ingestion owns one Job entity used by uploads, the cadastre load and the height computation
- Terrain3D stores computed heights on the parcel through Catalog

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
