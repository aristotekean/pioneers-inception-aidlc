# Requirements: Bogotá Urban Digital Twin (proof of concept)

Upstream: `intent-statement.md` (Intent Capture), `scope-document.md` and `intent-backlog.md` (Scope Definition), `team-practices.md` (Practices Discovery), `wireframes.md` and `user-flow.md` (Rough Mockups), `feasibility-assessment.md` and `constraint-register.md` (Feasibility). Answers: `requirements-analysis-questions.md`. Depth: Standard.

## Intent Analysis

Four researchers from a university, a private company and a public-sector agency want one web place to bring their own datasets and official Bogotá data together as layers on a map, explore them, and later run analyses such as flood risk. This piece of work is a proof of concept whose purpose is to validate the technical approach before further investment. Success is judged on three demonstrable outcomes: a researcher loads a dataset as a map layer without developer help; the official cadastre for all of Bogotá is visible and inspectable; terrain and building heights render in 3D for a pilot localidad. The flood model is exploratory and outside the measurable commitment.

Two decisions taken after Scope Definition shape these requirements: per-user login is now a must-have (practices interview, superseding "no login"), and private datasets may be admitted once per-layer visibility exists.

## Functional Requirements

### FR1. Accounts and access

- FR1.1 The system shall require a signed-in user for every screen and every data-changing request. [Q8]
- FR1.2 The system shall support exactly one role with administrator rights; every user may upload, style, delete their own layers and set each layer's visibility. [Q2]
- FR1.3 The system shall authenticate users with email and password; the initial four accounts are seeded by a developer at deployment, and no self-registration screen is required. [Q8]
- FR1.4 The system shall reject requests to the upload, delete and visibility operations from unauthenticated callers with an explicit error, never a silent failure. [Q8] team-practices

### FR2. Self-serve dataset upload

- FR2.1 The system shall accept CSV and JSON/GeoJSON files up to 10 MB through the "Agregar datos" wizard; larger files or other formats are refused with a message naming the limit and accepted formats. [Q3] [Q4] wireframes
- FR2.2 The system shall detect geometry in this order and show the result before loading: native geometry (GeoJSON), then coordinate columns (latitude/longitude, latitud/longitud, x/y); when none is found the wizard blocks progress and asks the user to choose the geometry column. intent-statement wireframes
- FR2.3 The system shall let the user name the layer and choose which columns are visible in the attribute popup before confirming. wireframes
- FR2.4 The system shall keep the raw uploaded file before processing it, for provenance. intent-statement (initial description, ingestion step 1)
- FR2.5 The system shall convert every ingested geometry to the map coordinate system (EPSG:4326) before storing it, treating official Bogotá inputs as MAGNA-SIRGAS / CTM12 (EPSG:9377) unless the file declares otherwise. intent-statement (initial description) constraint-register C-02
- FR2.6 The system shall run each load as a background job whose status (queued, running, done, failed) is visible in the layer list and the status bar. [Q6] wireframes
- FR2.7 The system shall record every row that fails to load, with its reason, on the job status and show the count to the user; a failed row is never dropped silently. team-practices (affirmed rule) [Q6]
- FR2.8 On successful load the system shall create the layer with a default style, list it in the layer panel, and zoom the map to its extent. user-flow

### FR3. Official cadastre

- FR3.1 The system shall provide a developer-run job that loads the official IDECA cadastral parcels for all of Bogotá (with number of floors where published) as a layer, converting from EPSG:9377. [Q3] intent-statement
- FR3.2 The cadastre layer shall be visible and inspectable on the map like any uploaded layer (toggle, popup, style). intent-statement scope-document
- FR3.3 The job shall be repeatable so a newer IDECA release replaces the layer without manual database edits. intent-statement (assumption, see Open Questions)

### FR4. Layer panel and map exploration

- FR4.1 The system shall list every layer the signed-in user may see, with a checkbox to toggle visibility on the map, a colour swatch and a style control. wireframes
- FR4.2 The system shall let the user change a layer's colour, opacity and point size, applying the change live. wireframes (assumption on styling scope)
- FR4.3 The system shall show, on a click on a feature, a popup anchored at the click with the layer's visible attributes; Escape or a click elsewhere closes it. wireframes user-flow
- FR4.4 The system shall show cursor coordinates, scale and running-job count in a status bar. wireframes
- FR4.5 The system shall let the uploader or any administrator delete a layer and all its stored data, after a confirmation. [Q7]
- FR4.6 The system shall let the uploader set each of their layers as visible to all users or to a chosen subset of users; a layer is hidden from users outside that set in the list, tiles and popups. [Q2] scope-document (per-layer visibility)
- FR4.7 The system shall show an empty state with guidance when the user has no visible layers. wireframes

### FR5. 3D view of the pilot localidad

- FR5.1 The system shall provide an explicit 2D / 3D toggle, 2D by default; switching keeps the current map position. wireframes user-flow
- FR5.2 In 3D the system shall render terrain from the IDECA digital terrain model and buildings extruded to their height (surface model minus terrain model) for the pilot localidad. intent-statement constraint-register C-03
- FR5.3 Outside the pilot localidad the 3D view shall show flat terrain and no buildings, with a banner explaining the pilot boundary. wireframes
- FR5.4 Uploaded and cadastre layers shall remain visible and inspectable in 3D. user-flow
- FR5.5 If terrain is unavailable the system shall fall back to 2D with a message. wireframes

### FR6. Operational visibility

- FR6.1 The system shall expose a health endpoint reporting the availability of the database and object storage. [Q6]
- FR6.2 The system shall emit structured error logs (timestamp, level, component, job or request identifier, message). [Q6]
- FR6.3 The system shall show per-job status and failure detail in the interface (FR2.6, FR2.7). [Q6]

### FR7. Interface

- FR7.1 The user interface shall be in Spanish. wireframes (rough-mockups Q8)
- FR7.2 The interface shall be usable on desktop and tablet browsers; touch equivalents for map navigation and the layer panel are required on tablet. wireframes (rough-mockups Q6, review finding R-01)

## Non-Functional Requirements

- NFR1 Map responsiveness: no pass/fail latency threshold is set for the proof of concept ("best effort"); the build shall measure and report the median and 95th-percentile tile response time for the cadastre layer at city scale under 4 concurrent users, so a threshold can be set later. [Q5]
- NFR2 Upload limits: files above 10 MB are refused before any processing starts; a 10 MB CSV load completes and the layer is visible within 5 minutes on the shared environment. [Q4] [assumption on duration]
- NFR3 Security: no credential, API key or secret in source code; secrets injected from the platform secret manager; passwords stored hashed with a current password-hashing algorithm; all traffic over HTTPS. team-practices (affirmed rule) [Q8]
- NFR4 Data protection: no personal data is ingested in the proof of concept; a dataset containing personal data requires a privacy impact assessment before ingestion. constraint-register C-11
- NFR5 Cost: the shared environment stays under USD 100 per month; rasters are limited to the pilot localidad until a separate cost decision. constraint-register C-09, C-03
- NFR6 Testability: test-first development for every layer, 80% line coverage, CI on GitHub Actions before merge, tests run against a disposable PostGIS container. team-practices
- NFR7 Observability: health endpoint (FR6.1), structured logs (FR6.2), job status (FR6.3). [Q6]
- NFR8 Accessibility: no formal conformance target for the proof of concept; keyboard entry points noted per screen in the wireframes are preserved. wireframes (rough-mockups Q7)
- NFR9 Availability: single shared environment, no redundancy requirement; an outage is acceptable for the proof of concept. team-practices (Deployment)

## Constraints

- C-A One or two developers full-time; target 3 to 6 months to the phase 1 to 4 demo. intent-statement constraint-register C-06, C-07
- C-B GCP preferred, account not yet confirmed; local development on Docker Compose. constraint-register C-10 team-practices
- C-C Official datasets published in EPSG:9377; all map data stored in EPSG:4326. constraint-register C-02
- C-D Pilot localidad only for terrain and surface rasters. constraint-register C-03
- C-E Walking skeleton first (thin upload-to-map slice), then the 3D pipeline, risk-first. scope-document team-practices
- C-F Affirmed hard rules: ALWAYS record per-row ingestion failures on the job status; NEVER hardcode credentials. team-practices

## Assumptions

- The pilot localidad is chosen at the start of the 3D unit; its terrain and surface products are downloadable without a special request. [assumption]
- Basic styling means colour, opacity and point size; richer symbology is not required. [assumption]
- A newer IDECA cadastre release replaces the whole layer rather than merging changes. [assumption]
- A 10 MB CSV load finishing within 5 minutes is achievable on the smallest shared environment. [assumption]

## Out of Scope

- Conversational query interface; orthophoto basemap. scope-document
- Geocoding by address or reference codes, and the external API connector (later backlog units PU-06, PU-08). intent-backlog
- The flood-risk model (exploratory, PU-09). intent-backlog
- Full-city rasters; self-registration; roles beyond the single administrator role. constraint-register C-03 [Q2] [Q8]

## Open Questions

- Which localidad is the pilot? To be decided before the 3D unit starts.
- Should layer visibility default to "all users" or "only me" on upload? Not asked; default proposed at design time.
- Does the cadastre job also load blocks (manzanas) and the official boundary layers needed later for reference-code geocoding? Deferred to the geocoding unit.
