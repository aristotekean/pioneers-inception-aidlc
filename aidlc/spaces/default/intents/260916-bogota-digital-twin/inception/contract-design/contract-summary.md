# Contract Summary: Bogotá Urban Digital Twin

Upstream: `unit-of-work.md` and `unit-of-work-dependency.md` (Units Generation), `components.md` (Domain Design), `requirements.md`. Answers: `contract-design-questions.md`. No API is consumed outside the system in this piece of work; every contract below is inter-unit. Common rules from the answers: bearer token in the Authorization header on every call except sign-in; one unversioned base path `/api`; additive-only changes, providers deploy first, consumers ignore unknown fields; one error body; job progress by polling; separate repositories per deployable.

## Contracts

| # | Provider Unit | Consumer | Mechanism | Owner |
|---|---|---|---|---|
| C1 | U2 identity | U8 web-shell (and every web unit through the shell) | JSON over HTTPS, bearer token | U2 |
| C2 | U3 catalog | U10 web-layers, U9 web-upload (visibility in step 3) | JSON over HTTPS | U3 |
| C3 | U3 catalog | U10 web-layers, U11 web-3d | Vector tiles over HTTPS served by the backend under row policies | U3 |
| C4 | U4 ingestion | U9 web-upload, U8 web-shell (job count) | JSON over HTTPS, multipart upload, polling | U4 |
| C5 | U7 jobs-terrain3d (through the backend) | U11 web-3d | JSON over HTTPS for locations; static tile files over HTTPS | U7 |
| C6 | U1 platform-core | U2, U3, U4, U6, U7 | Shared database schema and migrations, in-process service layers | U1 |
| C7 | U4 ingestion | U6 jobs-cadastre, U7 jobs-terrain3d | In-process service contract: Job lifecycle and coordinate conversion | U4 |
| C8 | U1 platform-core | operators, CI | Health endpoint | U1 |

The change to C3 versus the initial description: the backend serves tiles itself for the proof of concept (the simplest way, Q2), so U5 tile-serving becomes the specification of this endpoint and its latency measurement rather than a separate tile server configuration.

## Common Conventions

```yaml
# shared conventions applied by every HTTP contract
base_path: /api
auth:
  scheme: bearer
  header: Authorization: Bearer <token>
  token_lifetime_hours: 12
  unauthenticated: 401 with error code AUTH_REQUIRED
  forbidden: 403 with error code FORBIDDEN
error_body:
  code: string           # stable machine code, e.g. FILE_TOO_LARGE
  message: string        # Spanish, human readable
  details: object|null   # field errors, row numbers, limits
compatibility:
  versioning: none for the proof of concept
  changes: additive only; providers deploy first; consumers ignore unknown fields
  breaking_changes: not allowed in this piece of work
ids: UUID strings
timestamps: ISO 8601 UTC
coordinates: EPSG:4326, [longitude, latitude]
```

## C1: Identity (U2 to U8)

```yaml
openapi: 3.1.0
info: { title: Identity, version: 0.1.0 }
paths:
  /api/auth/login:
    post:
      summary: Sign in with email and password
      requestBody:
        content:
          application/json:
            schema:
              type: object
              required: [email, password]
              properties:
                email: { type: string, format: email }
                password: { type: string }
      responses:
        '200':
          description: Signed in
          content:
            application/json:
              schema:
                type: object
                properties:
                  token: { type: string }
                  expiresAt: { type: string, format: date-time }
                  user: { $ref: '#/components/schemas/User' }
        '401': { description: Wrong email or password; body code AUTH_FAILED, message never reveals which }
  /api/auth/logout:
    post:
      summary: End the session
      security: [{ bearer: [] }]
      responses: { '204': { description: Session ended } }
  /api/auth/me:
    get:
      summary: Current user
      security: [{ bearer: [] }]
      responses:
        '200': { description: Current user, content: { application/json: { schema: { $ref: '#/components/schemas/User' } } } }
        '401': { description: Missing or expired token }
components:
  securitySchemes:
    bearer: { type: http, scheme: bearer }
  schemas:
    User:
      type: object
      properties:
        id: { type: string }
        email: { type: string }
        displayName: { type: string }
```

Failure behaviour: five failed sign-ins for one email within 10 minutes return 429 with code TOO_MANY_ATTEMPTS (design default, marked assumption). Expired tokens return 401 with code TOKEN_EXPIRED so the shell can show "Tu sesión expiró".

## C2: Catalog (U3 to U10, U9)

```yaml
openapi: 3.1.0
info: { title: Catalog, version: 0.1.0 }
paths:
  /api/layers:
    get:
      summary: Layers visible to the current user
      responses:
        '200':
          content:
            application/json:
              schema: { type: array, items: { $ref: '#/components/schemas/Layer' } }
  /api/layers/{layerId}:
    get:
      summary: Layer detail including extent, style for the current user, visibility (owner only) and raw upload info
      responses:
        '200': { content: { application/json: { schema: { $ref: '#/components/schemas/LayerDetail' } } } }
        '403': { description: Not visible to the current user }
        '404': { description: Unknown layer }
    delete:
      summary: Delete the layer, its features, style and raw upload (owner only)
      responses:
        '204': { description: Deleted }
        '403': { description: Not the owner; code NOT_OWNER }
  /api/layers/{layerId}/features/{featureId}:
    get:
      summary: Visible attributes of one feature for the popup
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  id: { type: string }
                  layerId: { type: string }
                  properties: { type: object, additionalProperties: true }
        '403': { description: Layer not visible }
  /api/layers/{layerId}/style:
    put:
      summary: Set the current user's style for the layer
      requestBody:
        content:
          application/json:
            schema: { $ref: '#/components/schemas/Style' }
      responses: { '200': { description: Stored style } }
  /api/layers/{layerId}/visibility:
    put:
      summary: Set who can see the layer (owner only)
      requestBody:
        content:
          application/json:
            schema: { $ref: '#/components/schemas/Visibility' }
      responses:
        '200': { description: Applied immediately to list, tiles and attributes }
        '403': { description: Not the owner; code NOT_OWNER }
components:
  schemas:
    Layer:
      type: object
      properties:
        id: { type: string }
        name: { type: string }
        geometryType: { type: string, enum: [Point, LineString, Polygon, Raster] }
        featureCount: { type: integer }
        ownerUserId: { type: string }
        isOwner: { type: boolean }
        style: { $ref: '#/components/schemas/Style' }
        job: { $ref: '#/components/schemas/JobSummary' }
        createdAt: { type: string, format: date-time }
    LayerDetail:
      allOf:
        - $ref: '#/components/schemas/Layer'
        - type: object
          properties:
            extent: { type: array, items: { type: number }, minItems: 4, maxItems: 4 }
            visibleFields: { type: array, items: { type: string } }
            visibility: { $ref: '#/components/schemas/Visibility' }
            rawUpload:
              type: object
              properties:
                fileName: { type: string }
                sizeBytes: { type: integer }
                uploadedAt: { type: string, format: date-time }
    Style:
      type: object
      properties:
        color: { type: string, pattern: '^#[0-9a-fA-F]{6}$' }
        opacity: { type: number, minimum: 0, maximum: 1 }
        pointSize: { type: number, minimum: 1, maximum: 12 }
    Visibility:
      type: object
      required: [mode]
      properties:
        mode: { type: string, enum: [everyone, selected] }
        userIds: { type: array, items: { type: string } }
    JobSummary:
      type: object
      properties:
        id: { type: string }
        status: { type: string, enum: [queued, running, done, done_with_errors, failed] }
        rowsFailed: { type: integer }
```

## C3: Tiles (U3 to U10, U11)

```yaml
openapi: 3.1.0
info: { title: Tiles, version: 0.1.0 }
paths:
  /api/tiles/{layerId}/{z}/{x}/{y}.mvt:
    get:
      summary: Vector tile of one layer for the current user
      description: >
        Served by the backend inside the request transaction; the row policies
        keyed by the current user filter features and refuse layers the user
        may not see. Tiles are cacheable per user for 60 seconds (assumption).
      security: [{ bearer: [] }]
      parameters:
        - { name: layerId, in: path, required: true, schema: { type: string } }
        - { name: z, in: path, required: true, schema: { type: integer, minimum: 0, maximum: 22 } }
        - { name: x, in: path, required: true, schema: { type: integer } }
        - { name: y, in: path, required: true, schema: { type: integer } }
      responses:
        '200': { description: Mapbox Vector Tile, content: { application/vnd.mapbox-vector-tile: {} } }
        '204': { description: Empty tile }
        '401': { description: Missing token }
        '403': { description: Layer not visible; code FORBIDDEN }
        '404': { description: Unknown layer }
```

The map service must attach the bearer token to tile requests (the engine's tile loader supports request headers); U5 documents this endpoint, its cache rule and the latency measurement script (NFR1).

## C4: Ingestion and jobs (U4 to U9, U8)

```yaml
openapi: 3.1.0
info: { title: Ingestion, version: 0.1.0 }
paths:
  /api/uploads/inspect:
    post:
      summary: Read a file and report detected columns and geometry (wizard step 2), without loading
      requestBody:
        content:
          multipart/form-data:
            schema:
              type: object
              required: [file]
              properties:
                file: { type: string, format: binary }
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  format: { type: string, enum: [csv, geojson, json] }
                  rowCount: { type: integer }
                  columns: { type: array, items: { type: object, properties: { name: { type: string }, type: { type: string } } } }
                  geometry:
                    type: object
                    properties:
                      source: { type: string, enum: [geojson, latlon, wkt, none] }
                      latColumn: { type: string, nullable: true }
                      lonColumn: { type: string, nullable: true }
                      crs: { type: string, nullable: true }
                  warnings:
                    type: array
                    items: { type: object, properties: { code: { type: string }, count: { type: integer }, message: { type: string } } }
        '413': { description: Over 10 MB; code FILE_TOO_LARGE }
        '415': { description: Not CSV or JSON/GeoJSON; code UNSUPPORTED_FORMAT }
        '422': { description: Unreadable file; code FILE_UNREADABLE }
  /api/uploads:
    post:
      summary: Start a load job (wizard step 3)
      requestBody:
        content:
          multipart/form-data:
            schema:
              type: object
              required: [file, name, geometrySource]
              properties:
                file: { type: string, format: binary }
                name: { type: string, minLength: 1 }
                geometrySource: { type: string, enum: [geojson, latlon, wkt] }
                latColumn: { type: string }
                lonColumn: { type: string }
                crs: { type: string, default: 'EPSG:4326' }
                visibleFields: { type: array, items: { type: string } }
                visibility: { $ref: '#/components/schemas/Visibility' }
      responses:
        '202':
          description: Job accepted
          content:
            application/json:
              schema: { $ref: '#/components/schemas/Job' }
        '413': { description: Over 10 MB }
        '415': { description: Unsupported format }
        '422': { description: Missing geometry choice; code GEOMETRY_REQUIRED }
  /api/jobs:
    get:
      summary: Jobs of the current user, newest first; running count for the status bar
      responses:
        '200': { content: { application/json: { schema: { type: array, items: { $ref: '#/components/schemas/Job' } } } } }
  /api/jobs/{jobId}:
    get:
      summary: Job status; the web app polls this every 3 seconds while status is queued or running
      responses:
        '200': { content: { application/json: { schema: { $ref: '#/components/schemas/Job' } } } }
        '404': { description: Unknown or not the caller's job }
  /api/jobs/{jobId}/failures:
    get:
      summary: Failed rows with reasons, paged
      parameters:
        - { name: page, in: query, schema: { type: integer, default: 1 } }
        - { name: pageSize, in: query, schema: { type: integer, default: 100, maximum: 500 } }
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  total: { type: integer }
                  items:
                    type: array
                    items:
                      type: object
                      properties:
                        rowNumber: { type: integer }
                        reason: { type: string }
components:
  schemas:
    Job:
      type: object
      properties:
        id: { type: string }
        kind: { type: string, enum: [upload, cadastre_load, height_computation, terrain_tiles, tileset] }
        status: { type: string, enum: [queued, running, done, done_with_errors, failed] }
        layerId: { type: string, nullable: true }
        startedAt: { type: string, format: date-time, nullable: true }
        finishedAt: { type: string, format: date-time, nullable: true }
        rowsTotal: { type: integer }
        rowsLoaded: { type: integer }
        rowsFailed: { type: integer }
        message: { type: string, nullable: true }
        rawUpload:
          type: object
          nullable: true
          properties:
            fileName: { type: string }
            sizeBytes: { type: integer }
            uploadedAt: { type: string, format: date-time }
    Visibility:
      type: object
      properties:
        mode: { type: string, enum: [everyone, selected] }
        userIds: { type: array, items: { type: string } }
```

Failure behaviour: a job that cannot store the raw file fails before any feature is written (code STORAGE_UNAVAILABLE); a job interrupted by a runner restart is marked failed with code INTERRUPTED and may be started again by the user; a poll that times out is retried by the client with backoff, never treated as failure.

## C5: Terrain and 3D locations (U7 to U11)

```yaml
openapi: 3.1.0
info: { title: Terrain3D, version: 0.1.0 }
paths:
  /api/terrain:
    get:
      summary: Where the 3D view finds terrain and building tiles, and the pilot boundary
      security: [{ bearer: [] }]
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  pilot:
                    type: object
                    properties:
                      localidad: { type: string }
                      boundary: { type: object, description: GeoJSON Polygon in EPSG:4326 }
                  terrainTilesUrl: { type: string, nullable: true, description: base URL of quantized-mesh terrain tiles }
                  buildingsTilesetUrl: { type: string, nullable: true, description: URL of the 3D Tiles tileset.json }
                  generatedAt: { type: string, format: date-time, nullable: true }
        '404': { description: Terrain not yet produced; the view stays 2D with a message }
```

Tile files are static objects served over HTTPS from object storage through the backend's domain; they are public within the team (no per-layer visibility on rasters, assumption from Domain Design).

## C6: Shared schema (U1 to backend and job units)

```yaml
shared-schema:
  owner: u1-platform-core
  migrations: Alembic, one linear history, applied before any process starts
  tables:
    users: [id, email, password_hash, display_name, created_at]
    sessions: [id, user_id, token_hash, issued_at, expires_at]
    datasources: [id, name, kind, source_uri, source_crs, schema, loaded_at, owner_user_id]
    layers: [id, datasource_id, name, geometry_type, visible_fields, default_style, feature_count, owner_user_id, created_at]
    features: [id, layer_id, geom (EPSG:4326, GiST), properties (jsonb, GIN), height_m, valid_from, valid_to]
    visibility_grants: [id, layer_id, user_id (null when mode is everyone), mode]
    layer_styles: [id, layer_id, user_id, color, opacity, point_size]
    jobs: [id, kind, status, started_at, finished_at, rows_total, rows_loaded, rows_failed, owner_user_id, layer_id, message]
    job_row_failures: [id, job_id, row_number, reason]
    raw_uploads: [id, job_id, file_name, size_bytes, storage_uri, uploaded_at, checksum]
    raster_assets: [id, kind, localidad, storage_uri, resolution_m, source_crs, source_date, checksum]
    tilesets: [id, kind, localidad, storage_uri, generated_at, source_layer_id]
    scenarios: [id, model, parameters, input_layer_ids, result_layer_id, status, created_at]
  row_policies:
    principle: every read of layers, features, visibility_grants and layer_styles is filtered by the current user set by the backend for the request transaction (SET LOCAL app.user_id inside the transaction; never on a pooled connection outside one)
    visible_layer_rule: owner_user_id = current user OR a grant with mode everyone OR a grant naming the current user
    writes: only through the owning module's service layer; jobs and terrain modules never write features except through Catalog's service
  access_rule: modules read other modules' tables only through that module's service layer; the row policies are the single exception because they are enforced by the database itself
```

## C7: Job service contract (U4 to U6, U7)

```yaml
in-process-contract:
  provider: u4-ingestion
  consumers: [u6-jobs-cadastre, u7-jobs-terrain3d]
  operations:
    - start_job(kind, owner_user_id, layer_id?) -> Job
    - report_progress(job_id, rows_total, rows_loaded)
    - record_failure(job_id, row_number, reason)      # never drop a row silently
    - finish_job(job_id, status, message?)
    - convert_geometry(geometry, source_crs) -> geometry in EPSG:4326   # the single conversion routine
  guarantees:
    - record_failure is durable before finish_job returns
    - a job left running after a process crash is marked failed with INTERRUPTED at next runner start
```

## C8: Health (U1 to operators and CI)

```yaml
openapi: 3.1.0
info: { title: Health, version: 0.1.0 }
paths:
  /health:
    get:
      summary: Availability of the database and object storage; no authentication
      responses:
        '200': { description: All checks pass, body { status: ok, checks: { database: ok, storage: ok } } }
        '503': { description: A check failed, body names it }
```

## Contract Ownership Rules

- The provider unit owns each spec file and keeps it in its repository; consumers pin the spec they were built against and read it from the provider's repository at build time.
- Additive changes (new optional fields, new endpoints, new enum values consumers can ignore) are safe: providers deploy first, consumers ignore unknown fields.
- Breaking changes are not allowed in this piece of work; if one becomes unavoidable, the change is agreed by the four researchers and shipped as provider then consumer within the same day, with the old shape kept until the consumer is deployed.
- Error codes are part of the contract: adding a code is additive, renaming one is breaking.
- The shared schema (C6) is owned by platform-core; other units propose migrations by pull request to that repository.

## Open Questions

| Contract | Question | Blocks |
|---|---|---|
| C3 | Per-user tile caching for 60 seconds is a design default; confirm whether the map engine's cache keys must include the user to avoid cross-user reuse in a shared browser | U10, U11 |
| C5 | Whether terrain and 3D tile files are served through the backend domain (signed URLs) or directly from object storage with public-read within the team | U7, U11 |
| C1 | The rate limit on sign-in attempts (five per 10 minutes) is a design default to confirm | U2 |
| C6 | Whether `sessions` stores hashed tokens (proposed) or the backend issues stateless signed tokens with no table | U1, U2 |

## Assumptions & Open Questions

- The backend-served tile endpoint replaces the off-the-shelf tile server for the proof of concept; the initial description's tile server can return later behind the same URL shape. [assumption]
- Polling interval of 3 seconds and the 60-second per-user tile cache are design defaults. [assumption]
