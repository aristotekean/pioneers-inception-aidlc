**Collaborator:** aidlc-quality-agent

## Contribution

### 1. Testing Posture — concrete `team-practices.md` wording

I agree with the lead's test-after default and its targeted-regression addition, but the
draft leaves the two structured fields ("Methodology"/"Ordering") without a
spatial-risk qualifier and leaves the coverage floor entirely open. Proposed wording for
the interview to confirm or amend:

- **Methodology**: `custom` — test-after for application layers (ingestion parsing,
  API endpoints, map/layer-panel components), test-first for coordinate reprojection
  and geometry-detection logic specifically. This mixes cadences, so per the stage
  brief it must be recorded as `custom`, not `test-after`.
- **Ordering**: "Write the reprojection/geometry-detection tests before the
  implementation for that path; for every other testable layer, implement the layer
  then write and run that layer's tests before moving on."

Rationale: C-02 (silent CRS misplacement) and C-04 (no GIS/3D experience on the team)
are exactly the conditions test-first is for — a wrong reprojection produces plausible-
looking, silently wrong output that a test-after pass is unlikely to catch by
inspection. This is a refinement of the lead's already-open question, not a new one;
I'm proposing the resolution be recorded as `custom`, not left as a binary test-first-
vs-test-after choice.

### 2. Coverage floor — scope classification is the blocking gap

The lead correctly flags that `poc` (no floor) vs `mvp` (80% line-coverage + CI) is
unresolved. I cannot pick one for the team, but I recommend the interview separate two
questions rather than treat them as one:

1. Which workflow scope applies (this is a framework/process question the human
   answers directly).
2. Independent of that answer, should the CRS-reprojection and geometry-detection
   modules carry an 80% line-coverage floor *specifically*, enforced in CI, even if the
   overall project scope is `poc`? I recommend yes — these are the two highest-risk,
   least-team-experience modules (C-02, C-04), and a per-module floor is cheap for a
   1-2 person team to maintain (small modules) while the general-suite floor stays
   deferred until `mvp`.

### 3. Coverage tooling — Python/FastAPI backend

- **Test runner**: `pytest`, with `pytest-asyncio` for the async FastAPI endpoints and
  async ingestion workers.
- **API testing**: FastAPI's `TestClient` (sync) or `httpx.AsyncClient` (async) against
  the app instance — no live server needed for unit/integration tests.
- **Coverage**: `pytest-cov` (wraps `coverage.py`). If a floor is affirmed, enforce it
  with `pytest --cov=<package> --cov-fail-under=<N>` as a CI-blocking step, not just a
  report.
- **Database/PostGIS integration tests**: `testcontainers-python` with the
  `postgis/postgis` image (spins up a real PostGIS instance per test run, no shared
  state). This is more reliable than mocking the DB for reprojection/geometry code,
  since the correctness question is precisely "did this query/write land in the right
  CRS and table."
- **Geometry assertions**: use `shapely`/`pyproj` (or whatever geometry lib the
  implementation lands on) in test assertions to compare geometries by coordinate
  tolerance (e.g., `almost_equals` / distance-under-epsilon), never exact float
  equality — reprojection introduces floating-point drift that exact-equality tests
  would flag as false failures.

### 4. Coverage tooling — Angular/CesiumJS frontend

- **Test runner**: Angular CLI ships Karma/Jasmine by default; confirm with the team
  whether to keep the default or move to Jest (faster, no headless-browser
  requirement, easier to run in CI containers) — this is an open tooling choice, not
  yet decided anywhere in the evidence I reviewed.
- **Component testing**: Angular Testing Library (or `TestBed` directly) for the
  layer-panel and upload components.
- **CesiumJS-specific constraint**: Cesium's `Viewer` requires a WebGL context and
  real DOM measurements; it is impractical and slow to unit-test actual 3D rendering.
  Recommended pattern: wrap all Cesium calls behind a thin adapter/service
  (`CesiumMapService` or similar) so unit tests mock the adapter and assert on *what
  was asked of Cesium* (which entities/layers were added, which camera calls were
  made, which data was passed to `Cesium3DTileset`) rather than pixels rendered. Keep
  true rendering verification, if wanted at all, to a small number of manual or
  screenshot-diff smoke checks — not part of the unit/coverage floor.
- **Coverage**: Karma's built-in Istanbul integration (`karma-coverage`), or Jest's
  built-in coverage if the team switches runners.

### 5. Spatial-data-specific testing (this is the area most likely to be under-
specified in the interview, and where I have the most to add)

- **Coordinate conversion (EPSG:9377 → EPSG:4326)**: at minimum, one test per ingested
  dataset type asserting a known input coordinate reprojects to the expected output
  within a fixed tolerance (e.g., 1e-6 degrees), using 2-3 fixed reference points
  (documented, official IDECA coordinate pairs if available, otherwise
  independently-computed via `pyproj`/QGIS as the oracle — never assert against the
  code's own output). Add one round-trip property test (reproject forward then
  inverse-reproject; result should match the original within tolerance) to catch
  transform-direction and datum-mismatch bugs, which are the classic silent-failure
  mode here.
- **Geometry-detection paths**: the scope document and lead draft name four distinct
  detection paths (native geometry, lat/lon columns, reference-code join [UPZ/barrio/
  manzana], geocoded address). Each needs its own smoke test with a small fixture file
  (a handful of rows, not a full dataset) plus one negative test: an upload that
  matches none of the four paths should fail with a clear, surfaced error, not a
  silent partial import — this is a `construction.md` guardrail (errors must be
  surfaced, not swallowed) and directly testable.
- **Tile output (raster/3D pipeline — Martin, TiTiler, 3D Tiles)**: these are not
  pixel-deterministic outputs, so test structurally rather than by exact byte
  comparison: (a) a tile request for a known bounding box inside the pilot localidad
  returns HTTP 200 with the expected content-type (MVT for Martin, PNG/TIFF for
  TiTiler) and non-empty body; (b) a request outside the pilot localidad's bounds
  returns an empty/404 response rather than an error or (worse) full-city data; (c) one
  integration test asserting the 3D Tiles endpoint returns a valid tileset JSON with a
  bounding volume that matches the pilot localidad's known extent. These are
  integration tests against a seeded test PostGIS instance, not unit tests.
- **Fixture data**: recommend a small, versioned `test-fixtures/` directory (a handful
  of rows per geometry-detection path, one small raster/LiDAR sample clipped well
  below the pilot-localidad scale) checked into the repo, distinct from the real
  pilot-localidad dataset. This keeps CI fast and avoids depending on live IDECA
  service availability during test runs.

### 6. CI quality gates

No CI configuration exists yet (confirmed greenfield, per the lead's evidence file), so
this is fully open and should be part of the interview, not assumed:

- Which CI platform — GitHub Actions is the most common default and pairs well with
  the confirmed GCP/Cloud Build deployment target, but this is unconfirmed.
- Minimum gate regardless of scope: the existing suite must stay green (already
  implied by `org.md`'s `poc` floor) and lint/type-check must pass (devsecops/
  developer territory, noted here only for the handoff).
- If the per-module floor I proposed in §2 is accepted: `pytest --cov-fail-under`
  on the reprojection/geometry-detection package, blocking merge.
- PR-triggered run against a real (containerized) PostGIS instance for the geometry/
  tile integration tests — these cannot be meaningfully mocked without losing the
  thing that's actually being verified (correct spatial data landing correctly).

### 7. Additional gap: PostGIS/SQL migration testing

The lead's draft leaves PostGIS/SQL migration file naming as "fully open" but does not
raise migration *testing* at all. I recommend adding one CI smoke check: every
migration applies cleanly to a fresh PostGIS test database and (if the team adopts a
migration tool with rollback support) rolls back cleanly. This is cheap for a 1-2
person team and catches a class of bug (broken migration ordering, missing PostGIS
extension activation) that is otherwise only found in deployment.

## Positions

- AGREE: test-after as the general default, per `org.md`'s scope table and the lead's
  draft — no reason to diverge for the non-spatial layers.
- AGREE: the lead's proposal to keep the targeted CRS/geometry regression floor as an
  *addition* to (not a replacement for) the scope-determined floor.
- OBJECT: leaving the coverage-floor question as a single scope-classification
  decision undersells the risk — I recommend splitting it into the general-scope floor
  (human decision, framework-determined) and a per-module floor for reprojection/
  geometry-detection code that I recommend regardless of overall scope, for the
  reasons in §2.
- OBJECT: the draft's "Open for the interview" list under Testing Posture asks whether
  test-first is wanted for "reprojection/geometry-detection logic specifically" as a
  yes/no, but doesn't surface that a yes answer changes the recorded `Methodology`
  field to `custom` (per the stage brief's own instruction) rather than leaving it as
  `test-after` with a footnote — the interview question and the artifact format need
  to stay consistent.
- OBJECT: no CI platform, coverage-tooling choice, or PostGIS-test-database strategy
  appears anywhere in the lead's draft or discovered-rules — these are not covered by
  any existing `org.md` default, so they need to be raised as open interview questions
  rather than left implicit, or Construction will improvise them per-Bolt.
