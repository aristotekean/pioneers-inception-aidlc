# Architecture Decision Records: Domain Design

Upstream: `components.md`, `requirements.md`, `stories.md`, `team-practices.md`. Answers: `domain-design-questions.md`.

## ADR-001: Five backend building blocks plus one web application

- **Status**: Accepted (2026-09-16)
- **Context**: The initial description proposed catalog, ingestion and models. Since then per-user login and per-layer visibility became must-haves (FR1, FR4.6), the cadastre and height computations became developer-run jobs (US3.1, US5.6), and the affirmed practices require module boundaries through service layers.
- **Decision**: Identity, Catalog, Ingestion, Analysis (Q1) and a separate Terrain3D block (Q5), plus WebApp. Identity is separate so "who is the current user" is owned once; Terrain3D is separate because rasters, tiles and the height computation share nothing with row-oriented loads except the Job record.
- **Consequences**: Positive: each block has one owner for its entities and one change rate; the row-policy boundary sits in Catalog only; Terrain3D can be replaced or scaled independently. Negative: five blocks for a one-to-two-developer team means five service layers to keep honest; Terrain3D reuses Ingestion's Job, creating a dependency between the two job producers.
- **Alternatives Rejected**: Three blocks with Identity folded into Catalog (couples sessions to the registry). Terrain processing inside Ingestion (different skills, data and lifecycle; would inflate the block with the highest learning risk).

## ADR-002: Per-layer visibility enforced by database row policies

- **Status**: Accepted (2026-09-16)
- **Context**: The tile server reads feature tables directly with no custom code (initial description). Story US4.9 requires that a non-allowed user cannot fetch a private layer's tiles or attributes even by direct request. Two mechanisms were viable: a proxy in front of the tile server, or row-level policies in the database keyed by the current user.
- **Decision**: Row-level policies in the database (Q2). Every read of Feature, Layer and VisibilityGrant is filtered by the current user, so the tile server, the attribute lookup and the layer list all enforce the same rule with one definition.
- **Consequences**: Positive: one enforcement point; no custom tile code; the rule is testable in the disposable database used by the test suite. Negative: the tile server must run each request under the requesting user's identity, which off-the-shelf tile servers do not do by default; Contract Design must fix the mechanism (a per-request validated parameter to a database function, or per-user connections). Public layers and the seeded accounts must be part of every fixture.
- **Alternatives Rejected**: A TileAccess proxy component: simpler to reason about per request, but a second enforcement point that can drift from the list and attribute rules, plus a custom service in the hot path of every tile.

## ADR-003: One web application with a single map service

- **Status**: Accepted (2026-09-16)
- **Context**: The initial description keeps a map adapter "regardless" so the engine can change; the first answer to Q3 used the engine directly from every feature. The follow-up (Q6) resolved the contradiction.
- **Decision**: One application shell (no separate feature modules) whose features call a single map service that wraps the engine; no feature imports the engine directly. This supersedes neither the document's principle nor the shell answer: it keeps the adapter as one service and drops feature-module partitioning.
- **Consequences**: Positive: the engine remains replaceable behind one surface; a small team keeps one place for map behaviour; tests can stub the map service. Negative: the map service grows with every feature (layers, popups, camera, 3D); it needs discipline to stay a wrapper rather than a second application.
- **Alternatives Rejected**: Direct engine use from each feature (fast at first, couples every screen to the engine's API). Feature modules each with their own adapter (over-structured for six screens and two developers).

## ADR-004: One Job entity in Ingestion shared by all job kinds

- **Status**: Accepted (2026-09-16)
- **Context**: Uploads, the cadastre load and the height computation all run in the background and all must report per-row failures (affirmed hard rule) and status in the interface (FR6.3).
- **Decision**: Ingestion owns Job, JobRowFailure and RawUpload (Q4). Terrain3D and the cadastre load create Jobs through Ingestion's service layer.
- **Consequences**: Positive: the failure-reporting rule and the job detail dialog are implemented once. Negative: Terrain3D depends on Ingestion for bookkeeping; a change to the Job shape touches three producers.
- **Alternatives Rejected**: Per-job-type status records: three implementations of the same rule and three dialogs.

## ADR-005: Building heights stored on the parcel feature

- **Status**: Accepted (2026-09-16)
- **Context**: Heights are computed by Terrain3D (surface minus terrain) but read by the 3D view and shown in the popup as a parcel attribute (AC5.7.1, US3.2).
- **Decision**: Terrain3D writes the height through Catalog onto the parcel Feature (attribute heightM); parcels outside raster coverage keep an empty height, never zero (AC5.6.2).
- **Consequences**: Positive: one attribute lookup serves popup, extrusion and later analyses; no join across blocks at render time. Negative: Catalog's Feature carries an attribute owned by another block's computation; re-running the cadastre load must preserve or recompute heights.
- **Alternatives Rejected**: A separate BuildingHeight entity in Terrain3D (a join on every popup and tile; two owners of "what a parcel looks like").
