# Intent Capture Questions

## Sources

- [desc] Initial description: "build \n\n# Bogotá Urban Digital Twin — Technical Architecture\n\n**Version:** 0.1 (draft)\n**Date:** 2026-09-16\n**Status:** Proposed\n\n---\n\n## 1. Purpose and Scope\n\nThis document describes the proposed software architecture for a web-based urban digital twin of Bogotá, Colombia. It captures the requirements gathered so far, the key design decisions and their rationale, the target data sources, and a phased roadmap.\n\nThe system is intended as a research platform. Its first domain is **cadastre (catastro)**, and its first analytical use case is **flood-risk simulation** built on top of cadastral and terrain data.\n\n### 1.1 Functional requirements\n\n- Web-based system.\n- Ability to ingest external data sources (CSV, JSON, other databases, external APIs) and display them as layers on a map of Bogotá.\n- Visualization and exploration of layers (toggle, attribute inspection, styling).\n- Future: run simulations (initially flood risk) and visualize results as new layers.\n\n### 1.2 Context and constraints\n\n| Aspect | Value |\n|---|---|\n| Initial users | 4 researchers (internal; not a public-facing tool) |\n| Access control | Not required initially |\n| Data volume | Thousands of records per layer (initially) |\n| Ingestion mode | Batch uploads (CSV/JSON) and pulls from external APIs; streaming not needed initially |\n| Geometry in inputs | Unknown; must support explicit geometry (lat/lon, WKT, GeoJSON, shapefile) and reference-based geocoding (address, UPZ, barrio, manzana codes) |\n| Visualization | 2D now, 3D as an explicit goal |\n| Cloud | Undecided; GCP preferred |\n| Team stack | PostgreSQL, Python, Go, Angular; open to new technologies |\n\n---\n\n## 2. Architectural Principles\n\n1. **Modular monolith, not microservices.** For 4 users and thousands of records, a single well-structured backend with clear module boundaries is faster to build and easier to operate. The module boundaries (`catalog`, `ingestion`, `models`) are the seams along which services can be extracted later if needed.\n2. **PostGIS is the core.** All vector data, spatial indexing, reprojection and spatial analysis live in PostgreSQL + PostGIS.\n3. **Rasters live in object storage, not in the database.** Terrain and surface models are large; they are stored as Cloud Optimized GeoTIFFs and served/read on demand.\n4. **The map engine is isolated behind an abstraction.** The Angular frontend accesses the map through a `MapAdapter` interface so the rendering engine can evolve without rewriting application logic.\n5. **Analysis is a first-class module from day one.** Even though simulation is a later phase, the backend is structured so that a model \"receives layers + parameters and produces a result layer + scenario metadata\". This is what turns a GIS viewer into a digital twin.\n\n---\n\n## 3. System Overview\n\n```\n External sources                          Browser\n (CSV, JSON, APIs, IDECA/WFS)              (Angular + map engine)\n        │                                     │            │\n        ▼                                     ▼            ▼\n ┌─────────────┐    ┌──────────────┐   ┌──────────────┐  ┌──────────────────┐\n │  Ingestion  │    │     API      │   │ Vector tiles │  │ Raster / 3D tiles│\n │ (Py workers)│    │  (FastAPI)   │   │   (Martin)   │  │ (TiTiler, static)│\n └──────┬──────┘    └──────┬───────┘   └──────┬───────┘  └────────┬─────────┘\n        │                  │                  │                   │\n        ▼                  ▼                  ▼                   ▼\n ┌────────────────────────────────────┐   ┌──────────────────────────────┐\n │       PostgreSQL + PostGIS         │   │        Cloud Storage         │\n │  datasources / layers / features   │   │ raw uploads, COGs, 3D tiles  │\n └────────────────────────────────────┘   └──────────────────────────────┘\n```\n\n### 3.1 Components\n\n| Component | Technology | Responsibility |\n|---|---|---|\n| Frontend | Angular + CesiumJS (see §6) | Map, layer panel, attribute inspection, upload UI, scenario UI |\n| API | Python, FastAPI | Catalog CRUD, upload endpoints, attribute queries, feature detail, job status, scenario management |\n| Ingestion workers | Python (GeoPandas, Shapely, pyogrio/GDAL, Rasterio) | Asynchronous jobs: parse, infer schema, detect/geocode geometry, reproject, load |\n| Vector tile server | Martin (or pg_tileserv) | Serves MVT tiles directly from PostGIS with no custom code |\n| Raster tile server | TiTiler (FastAPI-based) | Serves COG rasters (DTM, DSM, orthophotos) as map tiles |\n| 3D tile generation | pg2b3dm | Generates 3D Tiles (LoD1 extruded buildings) from PostGIS polygons + height |\n| Database | PostgreSQL + PostGIS | Vector storage, catalog, spatial analysis |\n| Object storage | Google Cloud Storage | Raw uploaded files (provenance), COG rasters, generated tiles |\n| Analysis / simulation | Python module `models/`, run as jobs | Flood modelling and future models; writes result layers |\n\n### 3.2 Why Python over Go for the backend\n\nGo is a strong choice for high-throughput servers, but the geospatial and scientific ecosystem required by this project (GeoPandas, Shapely, GDAL/Rasterio, NumPy, hydrological libraries) is Python-native. Go remains an option for a specific high-performance service later; it is not needed for the initial system.\n\n---\n\n## 4. Data Model\n\nA generic model is used rather than one table per layer. At the expected volume this is simpler, requires no migrations per new dataset, and still performs well with a GiST index on geometry.\n\n```sql\nCREATE TABLE datasources (\n  id            uuid PRIMARY KEY,\n  name          text NOT NULL,\n  kind          text NOT NULL,          -- csv | json | api | wfs | shapefile | raster\n  source_uri    text,                   -- original URL or GCS path of raw file\n  source_crs    text,                   -- e.g. EPSG:9377\n  schema        jsonb,                  -- inferred columns and types\n  loaded_at     timestamptz,\n  created_at    timestamptz DEFAULT now()\n);\n\nCREATE TABLE layers (\n  id            uuid PRIMARY KEY,\n  datasource_id uuid REFERENCES datasources(id),\n  name          text NOT NULL,\n  geometry_type text,                   -- Point | LineString | Polygon | Raster\n  style         jsonb,                  -- default cartographic style\n  visible_fields text[],\n  created_at    timestamptz DEFAULT now()\n);\n\nCREATE TABLE features (\n  id            bigserial PRIMARY KEY,\n  layer_id      uuid REFERENCES layers(id),\n  geom          geometry(Geometry, 4326),\n  properties    jsonb NOT NULL DEFAULT '{}',\n  valid_from    timestamptz,\n  valid_to      timestamptz\n);\nCREATE INDEX features_geom_idx ON features USING gist (geom);\nCREATE INDEX features_layer_idx ON features (layer_id);\nCREATE INDEX features_props_idx ON features USING gin (properties);\n\nCREATE TABLE scenarios (                -- for the analysis module\n  id            uuid PRIMARY KEY,\n  model         text NOT NULL,          -- e.g. flood_v1\n  parameters    jsonb,\n  input_layers  uuid[],\n  result_layer  uuid REFERENCES layers(id),\n  status        text,\n  created_at    timestamptz DEFAULT now()\n);\n```\n\nNotes:\n\n- **CRS.** All geometry is stored in EPSG:4326. Official Bogotá datasets (IDECA, Catastro Distrital) are commonly published in MAGNA-SIRGAS / CTM12 (**EPSG:9377**); ingestion must always reproject.\n- **Temporal columns** (`valid_from`, `valid_to`) are included now at no cost so that the twin can hold historical states later.\n- Raster layers have a `layers` row pointing to a COG in Cloud Storage rather than rows in `features`.\n\n---\n\n## 5. Ingestion Pipeline\n\nUploading a file or registering an API source creates an asynchronous job (Cloud Run Jobs, or Celery/RQ locally). Steps:\n\n1. **Store raw input** in Cloud Storage for provenance.\n2. **Detect format** (CSV, JSON/GeoJSON, shapefile/GeoPackage, WFS, paginated REST API).\n3. **Infer schema** (column names, types) and record it on the `datasources` row.\n4. **Detect geometry**, in order of preference:\n   - Native geometry (GeoJSON, shapefile, WKT column).\n   - Coordinate columns (`lat`/`lon`, `latitud`/`longitud`, `x`/`y`).\n   - Reference codes: UPZ, localidad, barrio, manzana or cadastral ID → spatial join against official boundary layers loaded once from IDECA as base layers.\n   - Free-text address → geocoding (Google Geocoding API; performs well for Bogotá).\n5. **Reproject** to EPSG:4326.\n6. **Load** into `features`, create the `layers` row with a default style.\n7. **Report** job status and validation warnings (unmatched codes, failed geocodes) back to the API.\n\nThe API-source connector is configuration-driven (base URL, auth, pagination strategy, field mapping) so new external APIs can be added without code changes.\n\n---\n\n## 6. Visualization Strategy: 2D, 2.5D and 3D\n\n### 6.1 Distinguishing levels of \"3D\"\n\n| Level | What it is | Effort |\n|---|---|---|\n| 2D | Layers on a flat map | Baseline |\n| 2.5D (LoD1) | Building footprints extruded with real heights; terrain relief | Low, once heights are available |\n| 3D (LoD2+) | Real building models (CityGML/BIM), 3D Tiles, photorealistic tiles | High; different data formats and rendering engine |\n\n### 6.2 Decision\n\nTwo engines were evaluated for the frontend:\n\n- **MapLibre GL JS** — fastest path to a 2D/2.5D MVP. Supports extrusion and terrain natively. Ceiling: true 3D models and 3D Tiles require migrating to another engine.\n- **CesiumJS** — steeper learning curve and heavier client, but no ceiling. Natively supports terrain from a DEM, 3D Tiles, Google Photorealistic 3D Tiles, and ordinary 2D layers.\n\n**Decision: adopt CesiumJS from the start.** Rationale: 3D is an explicit goal, the flood use case benefits strongly from visualizing water depth against terrain and buildings, and the LiDAR-derived products already available for Bogotá (see §7) provide terrain and real building heights from day one. Starting with Cesium costs roughly two to three additional weeks at the beginning but avoids a mid-project frontend migration. The backend is identical under either choice.\n\nThe `MapAdapter` abstraction in Angular is kept regardless, to contain engine-specific code.\n\n### 6.3 3D data pipeline\n\n```\nCadastre footprints + heights (PostGIS) ──pg2b3dm──▶ 3D Tiles (GCS) ──▶ Cesium\nDTM raster (COG, GCS) ──terrain tiler──▶ quantized-mesh terrain tiles ──▶ Cesium\n```\n\nBuilding height per parcel is computed as **DSM − DTM** via zonal statistics (`rasterstats` or PostGIS raster) and stored in `properties.height_m`.\n\n---\n\n## 7. Official Data Sources: Cadastre and LiDAR\n\nBogotá's spatial data infrastructure (IDECA, Unidad Administrativa Especial de Catastro Distrital) publishes, under an open license, products that map directly onto the architecture:\n\n| Product | Type | Where it goes | Use |\n|---|---|---|---|\n| Cadastral parcels, blocks, constructions (with number of floors) | Vector (shapefile/GeoJSON/WFS) | PostGIS `features` | First datasource; base for extrusion and flood impact |\n| Digital Terrain Model (DTM/MDT), urban, 50 cm resolution (LiDAR 2020; a 2025 edition also listed) | Raster | GCS as COG | Terrain tiles; primary input for hydrological modelling |\n| Digital Surface Model (DSM/MDS), urban, LiDAR-derived | Raster | GCS as COG | Building heights (DSM − DTM) |\n| Orthophotos (8 cm GSD, 2020–2021) | Raster | GCS as COG | Optional imagery basemap |\n\nThese are derived products, not raw point clouds, which makes them far easier to use. Practical notes:\n\n- Verify the CRS on download; expect EPSG:9377.\n- Convert rasters to COG with GDAL (`gdal_translate -of COG`) before upload.\n- Develop the raster pipeline on a small clip (one localidad) before processing the full city.\n- Data are also exposed via OGC (WMS/WFS) and ESRI REST services, which the API-source connector can consume directly.\n\n---\n\n## 8. Analysis Module and Flood Risk\n\nNot built in the first phase, but the design accounts for it:\n\n- Backend module `models/` is separate from `ingestion/` and `catalog/`.\n- A model run is a job that reads input layers and rasters, executes, and writes a result layer plus a `scenarios` row (parameters, inputs, status).\n- Result layers are visualized like any other layer.\n- The DTM enables flow direction, flow accumulation, catchment delineation and ponding analysis; combined with precipitation inputs and the drainage network, and intersected with cadastral parcels, this yields affected-property estimates.\n- Established tools (e.g. HEC-RAS, LISFLOOD-FP, or simplified cellular models) can be wrapped as jobs.\n\n---\n\n## 9. Deployment (GCP)\n\n| Concern | Service |\n|---|---|\n| API, Martin, TiTiler, frontend | Cloud Run (containers) |\n| Ingestion and model jobs | Cloud Run Jobs |\n| Database | Cloud SQL for PostgreSQL with PostGIS |\n| Object storage | Cloud Storage |\n| Geocoding | Google Geocoding API |\n| Local development | Docker Compose with the same containers |\n\nGoogle Maps is used for geocoding only, not as the map engine (limited support for custom vector layers and extrusion; vendor lock-in). If photorealistic imagery is desired later, Google Photorealistic 3D Tiles can be consumed by Cesium.\n\n---\n\n## 10. Alignment with the Literature\n\nThe proposed design was reviewed against Xu et al. (2024), *Leveraging Generative AI for Urban Digital Twins* (arXiv:2405.19464), a scoping review.\n\n- **Architecture style.** The paper presents monolithic, microservice and edge architectures as valid options and describes the monolithic style as the foundation for integrating simulation systems and data sources in a client–server setting. The modular monolith proposed here is consistent with that.\n- **Layer mapping.** Against the paper's conceptual layers: the *Data* and *Service* layers are covered; the *Virtual* layer is covered at 2.5D/3D via §6; the *Cognition* layer (simulations, decision support) corresponds to the `models/` module in §8 and is what distinguishes a digital twin from a GIS viewer.\n- **Data enhancement.** The paper emphasises data quality, anomaly detection and enrichment. A validation step in the ingestion pipeline addresses this at a basic level.\n- **GenAI.** For monolithic architectures the paper recommends *GenAI as a Service* (external model APIs). The most practical application here is a conversational interface over the catalog (natural language → PostGIS queries), kept as an optional module.\n- **On 3D ambition.** The paper itself notes that building a comprehensive 3D city model for a large urban area is costly and labour-intensive, and cites work distinguishing city digital twins from 3D city models: the twin is data + models + feedback, not the rendering. Ambition is therefore placed in data integration and simulation, with 3D adopted where it adds communicative value (flood depth against buildings), using the LiDAR products that make LoD1 cheap.\n\n**Out of scope** (research directions in the literature, not requirements here): virtual reality, edge/IoT computing (no sensors), and GAN-based geometry generation.\n\n---\n\n## 11. Roadmap\n\n| Phase | Deliverable |\n|---|---|\n| 1 | PostGIS schema; manual load of IDECA cadastral parcels; Cesium map showing the layer in Angular |\n| 2 | Upload endpoint for CSV/JSON with lat/lon detection; layer catalog and layer panel (toggle, click-to-inspect) via Martin vector tiles |\n| 3 | DTM and DSM as COGs in GCS; TiTiler; Cesium terrain from the DTM |\n| 4 | Building heights (DSM − DTM); 3D Tiles via pg2b3dm; LoD1 city view |\n| 5 | Geocoding by address and by UPZ/barrio/manzana codes |\n| 6 | Configuration-driven connector for external APIs / WFS |\n| 7 | `models/` module and `scenarios`; first flood-risk model; result layers on the map |\n| Later | Conversational query interface (GenAI as a Service); access control if the tool opens to more users |\n\n---\n\n## 12. Open Questions\n\n- Which cloud provider is finally chosen (GCP assumed).\n- Whether third-party map imagery (Google) is acceptable versus fully open-source basemaps.\n- Team size and GIS experience available for the raster and 3D pipelines.\n- Target timeline for the MVP."
- [scope] Workflow-selected scope: `mvp`.

## Q1. What is the primary purpose of the Bogotá urban digital twin in its first version?

The description frames the system as a research platform whose first domain is cadastre and whose first analytical use case is flood-risk simulation. Everything downstream (requirements depth, access control, quality bar) depends on whether this is a research instrument or a decision tool.

A. A research platform for academic analysis (methods, experiments, publications)
B. A decision-support tool for city agencies or planners
C. Both: research first, with the explicit goal of becoming a decision-support tool
D. A proof of concept to validate the technical approach before committing further
E. Not yet defined
X. Other (please specify)

[Answer]: D. A proof of concept to validate the technical approach before committing further

## Q2. Who are the four initial researchers, organizationally, and what pain are they experiencing today?

The description says the initial users are four internal researchers. Their organization determines data access, hosting constraints, and who counts as the customer.

A. A university research group that today works with desktop GIS and scattered files
B. A public-sector team (district agency, IDECA/Catastro-adjacent) that lacks an integrated analysis platform
C. A private company or consultancy building internal capability
D. A mixed consortium (university plus public agency)
E. Not yet defined
X. Other (please specify)

[Answer]: X. Other — mixed: university, private company and a public-sector agency

## Q3. What measurable outcomes would show the first version succeeded? (select all that apply)

The description lists a roadmap but no success metrics. Measurable targets are required so later stages can verify them.

A. A new external dataset (CSV/JSON) can be loaded and shown as a map layer without developer intervention
B. Official IDECA cadastral parcels for all of Bogotá are visible and inspectable on the map
C. Terrain and LoD1 building heights are rendered in 3D from the LiDAR-derived products
D. A first flood-risk scenario produces a result layer that intersects with cadastral parcels
E. Not yet defined
X. Other (please specify)

[Answer]: A, B, C

## Q4. What is the trigger for starting this initiative now? (select all that apply)

Knowing the "why now" shapes priorities and the acceptable timeline.

A. A funded research project or grant with defined deliverables
B. The availability of open IDECA LiDAR products (DTM/DSM, orthophotos, cadastre) that make it feasible
C. Flood-risk pressure in Bogotá and demand for better analysis
D. An institutional opportunity or partnership (agency, university, funding body)
E. Not yet defined
X. Other (please specify)

[Answer]: A, B, D

## Q5. Beyond the four researchers, who are the key stakeholders and what does each care about? (select all that apply)

The description names no stakeholders outside the research team. Stakeholders are only recorded if you confirm them here.

A. A principal investigator or research lead accountable for outcomes
B. IDECA / Catastro Distrital as the official data provider
C. A city risk-management or planning agency as a future consumer of results
D. A funding body or sponsor expecting reports
E. None identified beyond the four researchers
X. Other (please specify)

[Answer]: B, C, D

## Q6. Who decides scope and priority for this project, and who influences those decisions?

A. A single research lead or principal investigator decides; the researchers influence
B. The four researchers decide by consensus
C. An external sponsor or agency decides; the research team influences
D. A technical lead decides on architecture; a research lead decides on features
E. Not yet defined
X. Other (please specify)

[Answer]: B. The four researchers decide by consensus

## Q7. Are there communication requirements or a reporting cadence the project must respect?

A. A weekly sync with the research team
B. Milestone-based demos at the end of each roadmap phase
C. Formal progress reports to a sponsor or funding body
D. Ad hoc communication only
E. None required
X. Other (please specify)

[Answer]: B. Milestone-based demos at the end of each roadmap phase

## Q8. The workflow was started with the "mvp" scope (build the core, skip operations). Does that match the product boundary you intend for this piece of work?

The mvp scope runs ideation, inception, and construction stages and skips deployment and operations stages. Confirm it, or define a different boundary.

A. Confirm mvp: the core is roadmap phases 1-2 (cadastral layer on the map, CSV/JSON upload, layer panel with vector tiles)
B. Different boundary: roadmap phases 1-4 (add terrain, building heights, and the LoD1 3D city view)
C. Different boundary: the full roadmap through phase 7 (first flood-risk model and result layers)
D. Different boundary: phase 1 only, as a thin proof of concept
E. Not yet defined
X. Other (please specify)

[Answer]: C. Different boundary: the full roadmap through phase 7 (first flood-risk model and result layers)

## Q9. Follow-up: how do "proof of concept" (Q1), "full roadmap through the flood model" (Q8) and success metrics without the flood scenario (Q3) fit together?

You said the purpose is a proof of concept (Q1: D), the product boundary is the full roadmap through phase 7 including the first flood-risk model (Q8: C), yet the success metrics you selected (Q3: A, B, C) stop at 3D terrain and do not include a flood scenario. A proof of concept whose boundary includes the flood model but whose success is judged without it is a contradiction later stages cannot plan around.

A. The boundary is the full roadmap, and a working flood-risk scenario on the map IS a success metric (add Q3 option D)
B. The boundary is the full roadmap, but the flood model is exploratory: success is judged on phases 1-4 only (keep Q3 as is)
C. Narrow the boundary of this piece of work to phases 1-4; the flood model becomes a separate later piece of work
D. Not yet defined
X. Other (please specify)

[Answer]: B. The boundary is the full roadmap, but the flood model is exploratory: success is judged on phases 1-4 only (keep Q3 as is)

## Q10. Follow-up: what is the target timeline for the first demonstrable milestone?

Your document lists the MVP timeline as an open question. Milestone demos (Q7) need a first date to aim at, and the timeline drives how much of the roadmap is realistic for a proof of concept.

A. About 1 to 2 months to the first demo (roadmap phase 1)
B. About 3 to 6 months to a demo covering phases 1 to 4
C. About 6 to 12 months to a demo covering the full roadmap
D. Not yet defined
X. Other (please specify)

[Answer]: B. About 3 to 6 months to a demo covering phases 1 to 4

## Consolidated Summary Confirmation

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
