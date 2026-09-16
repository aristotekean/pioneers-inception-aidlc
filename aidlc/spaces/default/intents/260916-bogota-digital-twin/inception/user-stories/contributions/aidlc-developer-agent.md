**Collaborator:** aidlc-developer-agent

## Contribution

Reviewed for implementability and sizing: is every story really 1 to 3 days for
one developer, does every acceptance criterion translate into an automated
test runnable against a disposable PostGIS container, and is anything a
developer would need missing.

### 1. US3.1 hides two vertical slices — split it

"Load the official cadastre by job" bundles (a) writing the developer-run job
that reuses the ingestion pipeline (geometry parse, reprojection from
US2.9, per-row failure handling) as a CLI/Cloud Run Job entry point, and (b)
acquiring, running against, and validating the real full-Bogotá IDECA parcel
file — a dataset with, per the architecture document, hundreds of thousands
of features, floor counts only "where published", and an unverified CRS
declaration that must be checked on download. (b) is data-wrangling and
scale validation, not code, and it cannot start until the file is in hand.
Together this is not 1 to 3 days. Proposed split:

- **US3.1 (narrowed): Build the developer-run cadastre job.** As Andrés, I
  want a developer-run job that reuses the upload pipeline's geometry
  parsing, reprojection and per-row failure recording to load a cadastral
  parcel file as a layer, so that the cadastre reaches the map the same way
  any other geometry does.
  - AC3.1.1 (kept)
  - AC3.1.3 (kept)
  - New AC3.1.4: Given a cadastre source file where 3 of its rows carry
    invalid or missing geometry, when the job runs, then the layer is
    created from the valid rows, the job status reports the 3 failed rows
    with their identifiers and reasons, and no row is dropped silently
    (matches the AC2.7.1 pattern and the project's Mandated hard rule — see
    Objection below).
  Depends on: US2.9. ~1 to 2 days.
- **New US3.1b: Load and validate the full-Bogotá cadastral dataset.** As
  Andrés, I want the cadastre job run against the real IDECA parcel file for
  all of Bogotá and its result checked against a known reference parcel, so
  that the shared map carries the real official base layer, not a sample.
  - AC3.1.2 (moved here, unchanged)
  - New: the job completes within an agreed time budget at full-city volume
    (numeric threshold to be set once the real file size is known).
  Depends on: US3.1 (narrowed), the new fixture story below. ~1 to 2 days,
  but the acquisition/verification step is schedule risk outside the
  developer's control (external file download, CRS confirmation), so it
  should not be pointed as pure coding time.

### 2. US5.2 bundles three independent pipelines — split it

"See terrain and building heights in the pilot localidad" is the single
story that carries the entire raster/3D pipeline the architecture document
lays out across two separate roadmap phases (§11, phases 3 and 4) and three
separate technology integrations (§6.3): a terrain tiler serving DTM COGs
to Cesium, a DSM-minus-DTM zonal-stats job producing `properties.height_m`,
and `pg2b3dm` generating 3D Tiles from the cadastre polygons plus that
height. Each is independently testable and independently risky (terrain
tile format, raster zonal-stats correctness, 3D Tiles generation), and each
is realistically 1 to 3 days on its own — together this is a 1-to-2-week
story mislabelled as one. Proposed split, in pipeline order:

- **US5.2a: Serve real terrain in 3D for the pilot localidad.** Terrain COG
  ingestion, terrain-tile serving, Cesium terrain rendering. Keeps
  AC5.2.1 (elevation within 2 m of a known point). Depends on: US5.1,
  US3.1 (narrowed), the new fixture story below.
- **US5.2b: Compute building heights for the pilot localidad.** The
  DSM-minus-DTM zonal-stats job that writes `properties.height_m` onto the
  cadastre parcels inside the pilot boundary. New AC: given a parcel with a
  known DSM and DTM value, when the job runs, then its
  `properties.height_m` is within 1 m of (DSM − DTM) for that parcel — this
  is the computation half of what AC5.2.2 currently asserts only at the
  rendering end. Depends on: US5.2a, US3.1b.
- **US5.2c: Extrude buildings to their computed height in 3D.** `pg2b3dm`
  tile generation and Cesium 3D Tiles rendering. Keeps AC5.2.2 (extrusion
  height within 1 m), now provable because US5.2b already guarantees the
  input height is correct. Depends on: US5.2b.

US5.3 and US5.4 should then depend on US5.2c rather than the single US5.2.

### 3. US4.6's tile-serving enforcement is an unresolved design question, not a 1–3 day story

AC4.6.2 requires a non-allowed user's direct tile request to be rejected.
The architecture document states Martin "serves MVT tiles directly from
PostGIS with no custom code" (§3.1) — that is its whole value proposition.
Enforcing an arbitrary per-layer, per-user allow-list at that layer means
either (a) a PostGIS Row-Level Security policy keyed on a session variable
Martin would have to be made to set per authenticated request, or (b) an
authorization check in front of Martin (API-side proxy or gateway), which
reintroduces the "custom code in front of the tile server" the architecture
chose to avoid. Neither option is decided anywhere in the artifacts I
reviewed. As scoped, US4.6 reads as a single 1–3 day story; the metadata
part (store an allow-list, filter the layer list, AC4.6.1/AC4.6.3) is that
size, but the tile/popup enforcement part (AC4.6.2) is blocked on an
architecture decision that hasn't been made. Recommend splitting into
US4.6 (metadata + list filtering) and a separate US4.6b (tile- and
popup-request enforcement) explicitly flagged as needing that decision
before it can be estimated or test-written.

### 4. Missing story: reference fixtures for the numeric acceptance criteria

Four acceptance criteria across three stories assert a numeric tolerance
against "a known reference point" / "a known parcel's centroid" / "a known
point's elevation" / "a known surface-minus-terrain height"
(AC2.9.1, AC3.1.2, AC5.2.1, AC5.2.2 as currently numbered). None of the
current stories owns producing that fixture data, and team.md's affirmed
testing posture requires these tests to run against small, versioned
fixtures checked into the repository, not a shared database. Without an
owning story, whichever story lands first has to invent this data ad hoc
and the fixture is not reused consistently. Proposed new story:

- **New story (Group 2 or a new "Foundations" group): Reference fixtures
  for spatial acceptance tests.** As a developer, I want small, versioned
  fixture files checked into the repository — a coordinate pair with a
  known EPSG:9377 source and its expected EPSG:4326 target, a mini cadastre
  extract with one parcel of known centroid and known floor count, and a
  small DTM/DSM raster clip with one point of known elevation and one
  parcel of known (DSM − DTM) height — so that the reprojection, cadastre
  and 3D acceptance tests are runnable and repeatable against the
  disposable PostGIS container without hand-built data per test run.
  Priority: Must Have (blocks the tests for US2.9, US3.1, US5.2a, US5.2b).
  Depends on: none. ~1 day.

### 5. Missing story: pilot localidad selection

requirements.md leaves "which localidad is the pilot?" as an open question
"to be decided before the 3D unit starts," and project.md's Corrections
record that the raster/3D pipeline is sequenced immediately after the
walking skeleton — but no story in Group 5 (or anywhere) owns making that
choice or acquiring the localidad-scoped IDECA terrain/surface downloads
C-D restricts rasters to. Without it, US5.2a has no defined input. Proposed
new story:

- **New story (Group 5, first in the group): Select the pilot localidad and
  acquire its terrain and surface clip.** As Andrés, I want the pilot
  localidad chosen and its IDECA DTM and DSM products downloaded, verified
  for CRS and coverage, and clipped to the localidad boundary, so that the
  3D pipeline has real, scoped input data to build against. Priority: Must
  Have. Traces: requirements Open Questions, C-D. Depends on: US3.1
  (narrowed). ~1 day of developer verification work, plus external download
  time outside developer control.

### 6. AC2.9 is missing the round-trip test team.md already commits to

team.md's Testing Posture requires, for coordinate reprojection, "one
round-trip test (reproject forward, then inverse-reproject) to catch
transform-direction and datum-mismatch bugs" — in addition to the
reference-point test. US2.9 has AC2.9.1 (forward test) but no round-trip
criterion. Proposed addition:

- New AC2.9.4: Given the known reference point from the fixture story
  above, when it is reprojected from EPSG:9377 to EPSG:4326 and then
  inverse-reprojected back to EPSG:9377, then the result is within the
  fixture's fixed tolerance of the original coordinate.

### 7. Sizing and testability notes on stories I did not change

- US1.1–US1.3, US2.1–US2.8, US2.10, US3.2–US3.3, US4.1–US4.5, US4.7, US5.1,
  US5.3–US5.5, US6.1–US6.2, US7.1–US7.2: each is a single vertical slice,
  each acceptance criterion names an observable outcome a test can assert
  against a disposable PostGIS container (or, for US7.x and the map/panel
  UI stories, a browser/component test) without a further hidden
  dependency. AC2.7.1's exact numbers (1,000 rows, 3 invalid, 997 features,
  "3 filas con error") are the right level of concreteness for a
  fixture-driven test — I used it as the template for the new AC3.1.4 above.
- US2.9's stated dependency on US2.6 (rather than the reverse) is
  implementation-plausible only if the walking skeleton's first working
  slice stores lat/lon CSV geometry as-is (no real reprojection) and adds
  EPSG:9377 handling afterward; the lead should state that explicitly in
  the story or Assumptions so a developer doesn't read US2.9 as blocking
  the walking skeleton.
- No story owns the initial Alembic/PostGIS schema (`datasources`, `layers`,
  `features`) from the architecture document. I am not proposing a story for
  it — it is enabling work that lands inside whichever story needs it first
  (US1.1 or US2.1), not a separate user-facing slice — but flagging it so it
  is not lost at Delivery Planning sizing.

## Positions

- OBJECT: US3.1 as written is not 1–3 days; it merges pipeline-code work
  with real-dataset acquisition and validation. Split per §1 above.
- OBJECT: US5.2 as written is not 1–3 days; it merges three independently
  testable pipelines (terrain, height computation, 3D tile extrusion) that
  the architecture document itself treats as separate roadmap phases. Split
  per §2 above.
- OBJECT: US4.6's AC4.6.2 (tile/popup enforcement) rests on an architecture
  decision not made in any artifact I read (RLS vs. a proxy in front of
  Martin) and should not be estimated as part of the same 1–3 day story as
  the metadata/list-filtering half. Split and flag per §3 above.
- OBJECT: no story owns the reference fixtures that AC2.9.1, AC3.1.2,
  AC5.2.1 and AC5.2.2 all depend on to be runnable tests; add the fixture
  story in §4.
- OBJECT: no story owns selecting the pilot localidad and acquiring its
  terrain/surface data, which US5.2a needs as an input and which
  requirements.md leaves as an open question; add the story in §5.
- OBJECT: AC2.9 is missing the round-trip reprojection test team.md's
  Testing Posture already commits to for every ingested dataset type; add
  AC2.9.4 per §6.
- AGREE: the remaining 1-to-3-day sizing, the feature-area grouping, and the
  Given/When/Then acceptance-criteria format are implementable as written
  and translate cleanly into tests against a disposable PostGIS container,
  per §7.
