# Units of Work: Bogotá Urban Digital Twin

Upstream: `components.md` and `decisions.md` (Domain Design), `requirements.md`, `stories.md`. Answers: `units-generation-questions.md` (grouped by deployable, split by capability, modular monolith, parallel work allowed, HTTP plus shared schema). This file defines what each unit is; `unit-of-work-dependency.md` defines what may depend on what; build order is chosen at Delivery Planning.

## Unit Table

| Unit ID | Directory | Kind | Deploys as | Complexity | Components |
|---|---|---|---|---|---|
| U1 | u1-platform-core | service | backend process | M | Catalog (schema), Ingestion (schema), Identity (schema) |
| U2 | u2-identity | service | backend process | S | Identity |
| U3 | u3-catalog | service | backend process | L | Catalog |
| U4 | u4-ingestion | service | backend process | L | Ingestion |
| U5 | u5-tile-serving | spec | tile server configuration | M | Catalog (row policies as consumed by the tile server) |
| U6 | u6-jobs-cadastre | service | job runner | M | Ingestion (cadastre job), Catalog |
| U7 | u7-jobs-terrain3d | service | job runner | XL | Terrain3D |
| U8 | u8-web-shell | ui | static frontend | M | WebApp |
| U9 | u9-web-upload | ui | static frontend | M | WebApp |
| U10 | u10-web-layers | ui | static frontend | L | WebApp |
| U11 | u11-web-3d | ui | static frontend | L | WebApp |

Deployment model per unit: U1 to U4 are modules of one backend process (shared deploy); U6 and U7 are modules of one job runner process (shared deploy); U8 to U11 are areas of one static frontend (shared deploy); U5 is configuration for the off-the-shelf vector tile server (standalone deploy of a third-party image). One database serves every backend and job unit.

## Unit Definitions

### U1 platform-core (service, M)
- **Boundary**: the backend skeleton every other backend unit plugs into: project layout with one package per building block and service-layer-only cross-module access, shared database schema and migrations, configuration and secrets injection, structured logging, health endpoint, the disposable test database and the reference fixture set.
- **Owns and delivers**: migrations for every entity in the catalogue (tables for all six blocks are created here so units add columns and policies, not tables), fixture loader, health endpoint (FR6.1), log format (FR6.2), CI wiring for lint, tests and coverage.
- **Notes**: no business behaviour; satisfies US6.1, US6.2, US9.1. The affirmed hard rules (no secrets in source, per-row failures recorded) are enforced structurally here: secret injection and the Job and JobRowFailure tables.

### U2 identity (service, S)
- **Boundary**: the Identity block: seeded accounts, sign-in, sessions of 12 hours, password hashing, unauthenticated rejection, HTTPS redirect.
- **Owns and delivers**: sign-in and sign-out endpoints, session middleware exposing the current user, seed command for the four accounts.
- **Notes**: US1.1 to US1.4. Every other backend unit reads the current user from this middleware.

### U3 catalog (service, L)
- **Boundary**: the Catalog block: datasources, layers, features, per-user styles, visibility grants, deletion with cascade, attribute lookup, the row policies keyed by the current user.
- **Owns and delivers**: layer list and detail endpoints filtered by visibility, attribute endpoint for popups, style endpoints, visibility endpoints (uploader only), delete endpoint, the row policies and the database function that sets the current user for a request.
- **Notes**: US2.10 data, US3.2 data, US4.1 to US4.3 data, US4.5, US4.6, US4.9; the tile server (U5) relies on the policies defined here. ADR-002 and the reviewer's note on pooled connections apply: the policy mechanism must be safe under connection pooling.

### U4 ingestion (service, L)
- **Boundary**: the Ingestion block: upload endpoint, size and format validation, geometry detection in order, the single coordinate conversion routine, background job execution, per-row failure recording, raw upload retention, job status and detail endpoints.
- **Owns and delivers**: US2.1 to US2.9, job status for US2.6 and US4.4, the conversion routine reused by U6 and U7.
- **Notes**: runs jobs through the job runner process but owns the code; NFR2 (10 MB within 5 minutes) is verified here.

### U5 tile-serving (spec, M)
- **Boundary**: the contract between the tile server and the database: server configuration, the function sources or views that expose feature tables under the row policies, how the current user reaches the database for a tile request, and the latency measurement script.
- **Owns and delivers**: tile server configuration checked into the repository, the tile access contract consumed by U10 and U11, the measurement report for NFR1 (US6.3).
- **Notes**: no application code beyond configuration and database functions; a spec unit, so it carries contracts and tests rather than business-logic design.

### U6 jobs-cadastre (service, M)
- **Boundary**: the cadastre load job: read the official parcel file in EPSG:9377, convert through U4's routine, write the parcel layer through U3, report per-row failures, repeatable atomic reload.
- **Owns and delivers**: US3.1, US3.3, US3.4; the parcel layer that U7 and U10 consume.
- **Notes**: runs in the job runner; full-city run needs the real IDECA release (US3.4).

### U7 jobs-terrain3d (service, XL)
- **Boundary**: the Terrain3D block: pilot localidad selection and data acquisition, raster asset registry, terrain tile production, height computation per parcel written through U3, 3D tile set generation.
- **Owns and delivers**: US9.2, US5.6, the terrain tiles and 3D tiles that U11 renders (US5.2, US5.7 data side).
- **Notes**: the highest learning risk; bounded to the pilot localidad by constraint C-D.

### U8 web-shell (ui, M)
- **Boundary**: the application shell: sign-in screen and client session handling, header with the 2D/3D control placeholder, side panel frame and tablet drawer, status bar, the single map service wrapping the engine, Spanish resource strings, base layout and theme.
- **Owns and delivers**: US1.1 and US1.2 (UI), US4.4, US4.8, US7.1, US7.2.
- **Notes**: every other web unit mounts inside this shell and calls the map service, never the engine.

### U9 web-upload (ui, M)
- **Boundary**: the add-data wizard (three steps, detection review, visibility in step 3, cancel flow) and the job detail dialog.
- **Owns and delivers**: US2.1 to US2.7 (UI), US2.11, job detail for US2.8.
- **Notes**: consumes U4's upload and job endpoints.

### U10 web-layers (ui, L)
- **Boundary**: the layer panel: list with toggles, style control, actions menu, feature popup, visibility dialog, delete confirmation, empty state, zoom to the new layer.
- **Owns and delivers**: US2.10, US3.2 (UI), US4.1 to US4.3, US4.5 to US4.7.
- **Notes**: consumes U3's endpoints and U5's tile contract through the map service.

### U11 web-3d (ui, L)
- **Boundary**: the 3D view: 2D/3D toggle behaviour, terrain and 3D tiles in the map service, pilot banner, loading banner, fallback to 2D, layers and popups in 3D.
- **Owns and delivers**: US5.1 to US5.5, US5.7 (view side).
- **Notes**: consumes U7's tile locations; the pilot boundary is read from U7's registry.

## Deferred

The later capabilities (US8.1 to US8.4: geocoding, reference codes, external connector, flood model) receive no unit in this piece of work; the Analysis block gets a unit when the flood model is planned. Recorded as deferred in `traceability.json`.

## Assumptions & Open Questions

- U5 is a spec because the tile server is an off-the-shelf image; if Contract Design finds the row-policy mechanism needs custom code, U5 becomes a service unit. [assumption]
- Complexity estimates are relative (S to XL) and inform Delivery Planning only. [assumption]
