# AI-DLC State Tracking

## Project Information
- **Project**: build # Bogotá Urban Digital Twin — Technical Architecture **Version:** 0.1 (draft) **Date:** 2026-09-16 **Status:** Proposed --- ## 1. Purpose and Scope This document describes the proposed software architecture for a web-based urban digital twin of Bogotá, Colombia. It captures the requirements gathered so far, the key design decisions and their rationale, the target data sources, and a phased roadmap. The system is intended as a research platform. Its first domain is **cadastre (catastro)**, and its first analytical use case is **flood-risk simulation** built on top of cadastral and terrain data. ### 1.1 Functional requirements - Web-based system. - Ability to ingest external data sources (CSV, JSON, other databases, external APIs) and display them as layers on a map of Bogotá. - Visualization and exploration of layers (toggle, attribute inspection, styling). - Future: run simulations (initially flood risk) and visualize results as new layers. ### 1.2 Context and constraints | Aspect | Value | |---|---| | Initial users | 4 researchers (internal; not a public-facing tool) | | Access control | Not required initially | | Data volume | Thousands of records per layer (initially) | | Ingestion mode | Batch uploads (CSV/JSON) and pulls from external APIs; streaming not needed initially | | Geometry in inputs | Unknown; must support explicit geometry (lat/lon, WKT, GeoJSON, shapefile) and reference-based geocoding (address, UPZ, barrio, manzana codes) | | Visualization | 2D now, 3D as an explicit goal | | Cloud | Undecided; GCP preferred | | Team stack | PostgreSQL, Python, Go, Angular; open to new technologies | --- ## 2. Architectural Principles 1. **Modular monolith, not microservices.** For 4 users and thousands of records, a single well-structured backend with clear module boundaries is faster to build and easier to operate. The module boundaries (`catalog`, `ingestion`, `models`) are the seams along which services can be extracted later if needed. 2. **PostGIS is the core.** All vector data, spatial indexing, reprojection and spatial analysis live in PostgreSQL + PostGIS. 3. **Rasters live in object storage, not in the database.** Terrain and surface models are large; they are stored as Cloud Optimized GeoTIFFs and served/read on demand. 4. **The map engine is isolated behind an abstraction.** The Angular frontend accesses the map through a `MapAdapter` interface so the rendering engine can evolve without rewriting application logic. 5. **Analysis is a first-class module from day one.** Even though simulation is a later phase, the backend is structured so that a model "receives layers + parameters and produces a result layer + scenario metadata". This is what turns a GIS viewer into a digital twin. --- ## 3. System Overview ``` External sources Browser (CSV, JSON, APIs, IDECA/WFS) (Angular + map engine) │ │ │ ▼ ▼ ▼ ┌─────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────────┐ │ Ingestion │ │ API │ │ Vector tiles │ │ Raster / 3D tiles│ │ (Py workers)│ │ (FastAPI) │ │ (Martin) │ │ (TiTiler, static)│ └──────┬──────┘ └──────┬───────┘ └──────┬───────┘ └────────┬─────────┘ │ │ │ │ ▼ ▼ ▼ ▼ ┌────────────────────────────────────┐ ┌──────────────────────────────┐ │ PostgreSQL + PostGIS │ │ Cloud Storage │ │ datasources / layers / features │ │ raw uploads, COGs, 3D tiles │ └────────────────────────────────────┘ └──────────────────────────────┘ ``` ### 3.1 Components | Component | Technology | Responsibility | |---|---|---| | Frontend | Angular + CesiumJS (see §6) | Map, layer panel, attribute inspection, upload UI, scenario UI | | API | Python, FastAPI | Catalog CRUD, upload endpoints, attribute queries, feature detail, job status, scenario management | | Ingestion workers | Python (GeoPandas, Shapely, pyogrio/GDAL, Rasterio) | Asynchronous jobs: parse, infer schema, detect/geocode geometry, reproject, load | | Vector tile server | Martin (or pg_tileserv) | Serves MVT tiles directly from PostGIS with no custom code | | Raster tile server | TiTiler (FastAPI-based) | Serves COG rasters (DTM, DSM, orthophotos) as map tiles | | 3D tile generation | pg2b3dm | Generates 3D Tiles (LoD1 extruded buildings) from PostGIS polygons + height | | Database | PostgreSQL + PostGIS | Vector storage, catalog, spatial analysis | | Object storage | Google Cloud Storage | Raw uploaded files (provenance), COG rasters, generated tiles | | Analysis / simulation | Python module `models/`, run as jobs | Flood modelling and future models; writes result layers | ### 3.2 Why Python over Go for the backend Go is a strong choice for high-throughput servers, but the geospatial and scientific ecosystem required by this project (GeoPandas, Shapely, GDAL/Rasterio, NumPy, hydrological libraries) is Python-native. Go remains an option for a specific high-performance service later; it is not needed for the initial system. --- ## 4. Data Model A generic model is used rather than one table per layer. At the expected volume this is simpler, requires no migrations per new dataset, and still performs well with a GiST index on geometry. ```sql CREATE TABLE datasources ( id uuid PRIMARY KEY, name text NOT NULL, kind text NOT NULL, -- csv | json | api | wfs | shapefile | raster source_uri text, -- original URL or GCS path of raw file source_crs text, -- e.g. EPSG:9377 schema jsonb, -- inferred columns and types loaded_at timestamptz, created_at timestamptz DEFAULT now() ); CREATE TABLE layers ( id uuid PRIMARY KEY, datasource_id uuid REFERENCES datasources(id), name text NOT NULL, geometry_type text, -- Point | LineString | Polygon | Raster style jsonb, -- default cartographic style visible_fields text[], created_at timestamptz DEFAULT now() ); CREATE TABLE features ( id bigserial PRIMARY KEY, layer_id uuid REFERENCES layers(id), geom geometry(Geometry, 4326), properties jsonb NOT NULL DEFAULT '{}', valid_from timestamptz, valid_to timestamptz ); CREATE INDEX features_geom_idx ON features USING gist (geom); CREATE INDEX features_layer_idx ON features (layer_id); CREATE INDEX features_props_idx ON features USING gin (properties); CREATE TABLE scenarios ( -- for the analysis module id uuid PRIMARY KEY, model text NOT NULL, -- e.g. flood_v1 parameters jsonb, input_layers uuid[], result_layer uuid REFERENCES layers(id), status text, created_at timestamptz DEFAULT now() ); ``` Notes: - **CRS.** All geometry is stored in EPSG:4326. Official Bogotá datasets (IDECA, Catastro Distrital) are commonly published in MAGNA-SIRGAS / CTM12 (**EPSG:9377**); ingestion must always reproject. - **Temporal columns** (`valid_from`, `valid_to`) are included now at no cost so that the twin can hold historical states later. - Raster layers have a `layers` row pointing to a COG in Cloud Storage rather than rows in `features`. --- ## 5. Ingestion Pipeline Uploading a file or registering an API source creates an asynchronous job (Cloud Run Jobs, or Celery/RQ locally). Steps: 1. **Store raw input** in Cloud Storage for provenance. 2. **Detect format** (CSV, JSON/GeoJSON, shapefile/GeoPackage, WFS, paginated REST API). 3. **Infer schema** (column names, types) and record it on the `datasources` row. 4. **Detect geometry**, in order of preference: - Native geometry (GeoJSON, shapefile, WKT column). - Coordinate columns (`lat`/`lon`, `latitud`/`longitud`, `x`/`y`). - Reference codes: UPZ, localidad, barrio, manzana or cadastral ID → spatial join against official boundary layers loaded once from IDECA as base layers. - Free-text address → geocoding (Google Geocoding API; performs well for Bogotá). 5. **Reproject** to EPSG:4326. 6. **Load** into `features`, create the `layers` row with a default style. 7. **Report** job status and validation warnings (unmatched codes, failed geocodes) back to the API. The API-source connector is configuration-driven (base URL, auth, pagination strategy, field mapping) so new external APIs can be added without code changes. --- ## 6. Visualization Strategy: 2D, 2.5D and 3D ### 6.1 Distinguishing levels of "3D" | Level | What it is | Effort | |---|---|---| | 2D | Layers on a flat map | Baseline | | 2.5D (LoD1) | Building footprints extruded with real heights; terrain relief | Low, once heights are available | | 3D (LoD2+) | Real building models (CityGML/BIM), 3D Tiles, photorealistic tiles | High; different data formats and rendering engine | ### 6.2 Decision Two engines were evaluated for the frontend: - **MapLibre GL JS** — fastest path to a 2D/2.5D MVP. Supports extrusion and terrain natively. Ceiling: true 3D models and 3D Tiles require migrating to another engine. - **CesiumJS** — steeper learning curve and heavier client, but no ceiling. Natively supports terrain from a DEM, 3D Tiles, Google Photorealistic 3D Tiles, and ordinary 2D layers. **Decision: adopt CesiumJS from the start.** Rationale: 3D is an explicit goal, the flood use case benefits strongly from visualizing water depth against terrain and buildings, and the LiDAR-derived products already available for Bogotá (see §7) provide terrain and real building heights from day one. Starting with Cesium costs roughly two to three additional weeks at the beginning but avoids a mid-project frontend migration. The backend is identical under either choice. The `MapAdapter` abstraction in Angular is kept regardless, to contain engine-specific code. ### 6.3 3D data pipeline ``` Cadastre footprints + heights (PostGIS) ──pg2b3dm──▶ 3D Tiles (GCS) ──▶ Cesium DTM raster (COG, GCS) ──terrain tiler──▶ quantized-mesh terrain tiles ──▶ Cesium ``` Building height per parcel is computed as **DSM − DTM** via zonal statistics (`rasterstats` or PostGIS raster) and stored in `properties.height_m`. --- ## 7. Official Data Sources: Cadastre and LiDAR Bogotá's spatial data infrastructure (IDECA, Unidad Administrativa Especial de Catastro Distrital) publishes, under an open license, products that map directly onto the architecture: | Product | Type | Where it goes | Use | |---|---|---|---| | Cadastral parcels, blocks, constructions (with number of floors) | Vector (shapefile/GeoJSON/WFS) | PostGIS `features` | First datasource; base for extrusion and flood impact | | Digital Terrain Model (DTM/MDT), urban, 50 cm resolution (LiDAR 2020; a 2025 edition also listed) | Raster | GCS as COG | Terrain tiles; primary input for hydrological modelling | | Digital Surface Model (DSM/MDS), urban, LiDAR-derived | Raster | GCS as COG | Building heights (DSM − DTM) | | Orthophotos (8 cm GSD, 2020–2021) | Raster | GCS as COG | Optional imagery basemap | These are derived products, not raw point clouds, which makes them far easier to use. Practical notes: - Verify the CRS on download; expect EPSG:9377. - Convert rasters to COG with GDAL (`gdal_translate -of COG`) before upload. - Develop the raster pipeline on a small clip (one localidad) before processing the full city. - Data are also exposed via OGC (WMS/WFS) and ESRI REST services, which the API-source connector can consume directly. --- ## 8. Analysis Module and Flood Risk Not built in the first phase, but the design accounts for it: - Backend module `models/` is separate from `ingestion/` and `catalog/`. - A model run is a job that reads input layers and rasters, executes, and writes a result layer plus a `scenarios` row (parameters, inputs, status). - Result layers are visualized like any other layer. - The DTM enables flow direction, flow accumulation, catchment delineation and ponding analysis; combined with precipitation inputs and the drainage network, and intersected with cadastral parcels, this yields affected-property estimates. - Established tools (e.g. HEC-RAS, LISFLOOD-FP, or simplified cellular models) can be wrapped as jobs. --- ## 9. Deployment (GCP) | Concern | Service | |---|---| | API, Martin, TiTiler, frontend | Cloud Run (containers) | | Ingestion and model jobs | Cloud Run Jobs | | Database | Cloud SQL for PostgreSQL with PostGIS | | Object storage | Cloud Storage | | Geocoding | Google Geocoding API | | Local development | Docker Compose with the same containers | Google Maps is used for geocoding only, not as the map engine (limited support for custom vector layers and extrusion; vendor lock-in). If photorealistic imagery is desired later, Google Photorealistic 3D Tiles can be consumed by Cesium. --- ## 10. Alignment with the Literature The proposed design was reviewed against Xu et al. (2024), *Leveraging Generative AI for Urban Digital Twins* (arXiv:2405.19464), a scoping review. - **Architecture style.** The paper presents monolithic, microservice and edge architectures as valid options and describes the monolithic style as the foundation for integrating simulation systems and data sources in a client–server setting. The modular monolith proposed here is consistent with that. - **Layer mapping.** Against the paper's conceptual layers: the *Data* and *Service* layers are covered; the *Virtual* layer is covered at 2.5D/3D via §6; the *Cognition* layer (simulations, decision support) corresponds to the `models/` module in §8 and is what distinguishes a digital twin from a GIS viewer. - **Data enhancement.** The paper emphasises data quality, anomaly detection and enrichment. A validation step in the ingestion pipeline addresses this at a basic level. - **GenAI.** For monolithic architectures the paper recommends *GenAI as a Service* (external model APIs). The most practical application here is a conversational interface over the catalog (natural language → PostGIS queries), kept as an optional module. - **On 3D ambition.** The paper itself notes that building a comprehensive 3D city model for a large urban area is costly and labour-intensive, and cites work distinguishing city digital twins from 3D city models: the twin is data + models + feedback, not the rendering. Ambition is therefore placed in data integration and simulation, with 3D adopted where it adds communicative value (flood depth against buildings), using the LiDAR products that make LoD1 cheap. **Out of scope** (research directions in the literature, not requirements here): virtual reality, edge/IoT computing (no sensors), and GAN-based geometry generation. --- ## 11. Roadmap | Phase | Deliverable | |---|---| | 1 | PostGIS schema; manual load of IDECA cadastral parcels; Cesium map showing the layer in Angular | | 2 | Upload endpoint for CSV/JSON with lat/lon detection; layer catalog and layer panel (toggle, click-to-inspect) via Martin vector tiles | | 3 | DTM and DSM as COGs in GCS; TiTiler; Cesium terrain from the DTM | | 4 | Building heights (DSM − DTM); 3D Tiles via pg2b3dm; LoD1 city view | | 5 | Geocoding by address and by UPZ/barrio/manzana codes | | 6 | Configuration-driven connector for external APIs / WFS | | 7 | `models/` module and `scenarios`; first flood-risk model; result layers on the map | | Later | Conversational query interface (GenAI as a Service); access control if the tool opens to more users | --- ## 12. Open Questions - Which cloud provider is finally chosen (GCP assumed). - Whether third-party map imagery (Google) is acceptable versus fully open-source basemaps. - Team size and GIS experience available for the raster and 3D pipelines. - Target timeline for the MVP.
- **Project Description Source**: project-description.json
- **Project Type**: Greenfield
- **Scope**: mvp
- **Start Date**: 2026-09-16T15:47:25Z
- **State Version**: 8
- **Active Agent**: aidlc-pipeline-deploy-agent
- **Worktree Path**:
- **Bolt Refs**:
- **Practices Affirmed Timestamp**:

## Scope Configuration
- **Stages to Execute**: 0.1, 0.2, 0.3, 1.1, 1.3, 1.4, 1.6, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 2.8, 2.9, 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7
- **Stages to Skip**: 1.2 (market-research), 1.5 (team-formation), 1.7 (approval-handoff), 4.1 (deployment-pipeline), 4.2 (environment-provisioning), 4.3 (deployment-execution), 4.4 (observability-setup), 4.5 (incident-response), 4.6 (performance-validation), 4.7 (feedback-optimization), 2.1 (reverse-engineering — greenfield)
- **Depth**: Standard
- **Test Strategy**: Standard
- **Review Override**: 
- **Change Control**: relaxed (from scope mvp)
- **Sensors**: on (from scope mvp)
- **Learnings**: on (from scope mvp)
- **Summary Confirmation**: on (from scope mvp)

## Workspace State
- **Project Root**: .
- **Languages**: Unknown
- **Frameworks**: Unknown
- **Build System**: Unknown

## Execution Plan Summary
- **Total Stages**: 22
- **Completed**: 7
- **In Progress**: practices-discovery

## Runtime State
- **Revision Count**: 0

## Phase Progress
<!-- Status values: Pending, Active, Verified, Skipped -->

- **Initialization**: Verified
- **Ideation**: Verified
- **Inception**: Active
- **Construction**: Pending
- **Operation**: Skipped

## Stage Progress
<!-- Checkbox states: [ ] not started, [-] in progress, [?] awaiting approval (gate open), [R] revising (user rejected gate), [x] completed, [S] skipped via --stage/--phase jump -->

### INITIALIZATION PHASE
- [x] workspace-scaffold — EXECUTE
- [x] workspace-detection — EXECUTE
- [x] state-init — EXECUTE

### IDEATION PHASE
- [x] intent-capture — EXECUTE
- [ ] market-research — SKIP
- [x] feasibility — EXECUTE
- [x] scope-definition — EXECUTE
- [ ] team-formation — SKIP
- [x] rough-mockups — EXECUTE
- [ ] approval-handoff — SKIP

### INCEPTION PHASE
- [ ] reverse-engineering — SKIP
- [?] practices-discovery — EXECUTE
- [ ] requirements-analysis — EXECUTE
- [ ] user-stories — EXECUTE
- [ ] refined-mockups — EXECUTE
- [ ] domain-design — EXECUTE
- [ ] units-generation — EXECUTE
- [ ] contract-design — EXECUTE
- [ ] delivery-planning — EXECUTE

### CONSTRUCTION PHASE
Per unit: [TBD]
- [ ] functional-design — EXECUTE
- [ ] nfr-requirements — EXECUTE
- [ ] nfr-design — EXECUTE
- [ ] infrastructure-design — EXECUTE
- [ ] code-generation — EXECUTE
- [ ] build-and-test — EXECUTE
- [ ] ci-pipeline — EXECUTE

### OPERATION PHASE
- [ ] deployment-pipeline — SKIP
- [ ] environment-provisioning — SKIP
- [ ] deployment-execution — SKIP
- [ ] observability-setup — SKIP
- [ ] incident-response — SKIP
- [ ] performance-validation — SKIP
- [ ] feedback-optimization — SKIP

## Current Status
- **Lifecycle Phase**: INCEPTION
- **Current Stage**: practices-discovery
- **Next Stage**: requirements-analysis
- **Status**: Running
- **Last Updated**: 2026-09-16T19:42:01Z

## Session Resume Point
- **Last Completed Stage**: rough-mockups
- **Next Action**: Execute Practices Discovery
- **Pending Artifacts**: none
