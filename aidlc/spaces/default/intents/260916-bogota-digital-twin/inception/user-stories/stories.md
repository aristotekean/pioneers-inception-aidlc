# User Stories: Bogotá Urban Digital Twin

Upstream: `requirements.md` (FR/NFR IDs), `team-practices.md` (test-first: every acceptance criterion becomes a test before its code), `personas.md`. Grouped by feature area (plan Q2), fine-grained (plan Q3), later capabilities at title level (plan Q4). Priorities are MoSCoW; the MVP boundary is decided at Delivery Planning. Integrated after the designer, developer and quality engineer reviewed the draft (see `contributions/`).

## Group 1: Accounts and access (FR1)

### US1.1 Sign in with email and password
As Andrés, I want to sign in with my email and password, so that only the four of us can reach the map and its data.
Priority: Must Have. Traces: FR1.1, FR1.3. Depends on: none.
- AC1.1.1 Given a seeded account, when I enter its email and correct password on the "Iniciar sesión" screen, then I land on the map view signed in.
- AC1.1.2 Given a seeded account, when I enter a wrong password, then I stay on the sign-in screen with the message "Correo o contraseña incorrectos", which does not reveal whether the email exists.
- AC1.1.3 Given I am not signed in, when I open any application URL, then I am redirected to the sign-in screen.
- AC1.1.4 Given a tablet viewport, when the sign-in screen renders, then its fields and button are at least 44 by 44 pixels and labelled in Spanish.
INVEST: independent, small (1 to 2 days), testable through the sign-in endpoint and a browser test.

### US1.2 Stay signed in and sign out
As Camila, I want my session to persist while I work and to end when I sign out, so that I do not re-enter my password on every action.
Priority: Must Have. Traces: FR1.1. Depends on: US1.1.
- AC1.2.1 Given I signed in, when I reload the page within 12 hours, then I remain signed in.
- AC1.2.2 Given I signed in more than 12 hours ago, when I make a data request, then I am redirected to the sign-in screen.
- AC1.2.3 Given I signed in, when I click "Cerrar sesión", then I am returned to the sign-in screen and a subsequent data request is rejected.

### US1.3 Reject unauthenticated data changes
As Andrés, I want every upload, delete and visibility request to be rejected without a session, so that nobody outside the team can change our data.
Priority: Must Have. Traces: FR1.4. Depends on: US1.1.
- AC1.3.1 Given no session, when a client calls the upload, delete or visibility endpoint, then the response is an explicit authentication error and nothing is written.
- AC1.3.2 Given no session, when a client requests tiles or attributes of any layer, then the response is an explicit authentication error.

### US1.4 Store passwords safely and serve over HTTPS
As Andrés, I want passwords stored only as strong hashes and every request served over HTTPS, so that a leaked database or an eavesdropper cannot recover credentials.
Priority: Must Have. Traces: NFR3. Depends on: US1.1.
- AC1.4.1 Given a seeded account, when I read its row in the database, then the stored value is a salted hash from a current password-hashing algorithm and the plain password appears nowhere.
- AC1.4.2 Given the deployed environment, when a client requests any URL over plain HTTP, then it is redirected to HTTPS.
- AC1.4.3 Given the repository, when the secret scanner runs in CI, then no credential or key is found in source.

## Group 2: Self-serve upload (FR2)

### US2.1 Choose a file within the accepted formats and size
As Camila, I want the wizard to accept my CSV or GeoJSON file up to 10 MB and refuse anything else clearly, so that I know immediately whether my file can be loaded.
Priority: Must Have. Traces: FR2.1. Depends on: US1.1.
- AC2.1.1 Given a 4 MB CSV, when I choose it in step 1, then the wizard accepts it and enables "Siguiente".
- AC2.1.2 Given a 12 MB CSV, when I choose it, then the wizard refuses it with a message naming the 10 MB limit and no upload starts.
- AC2.1.3 Given a zipped shapefile, when I choose it, then the wizard refuses it with a message naming CSV and JSON/GeoJSON as accepted formats.
- AC2.1.4 Given a tablet without drag-and-drop, when step 1 renders, then the "Elegir CSV o JSON" file picker alone is sufficient to choose a file.

### US2.2 Detect native geometry in GeoJSON
As Andrés, I want the wizard to recognise the geometry in a GeoJSON file, so that I do not have to describe columns for files that already carry shapes.
Priority: Must Have. Traces: FR2.2. Depends on: US2.1.
- AC2.2.1 Given a valid GeoJSON FeatureCollection, when step 2 opens, then "GeoJSON" is pre-selected as the geometry source and the feature count is shown.
- AC2.2.2 Given a GeoJSON file with a feature lacking geometry, when step 2 opens, then the count of features without geometry is shown as a warning and the load can still proceed.
- AC2.2.3 Given a GeoJSON file where no feature has a valid geometry, when step 2 opens, then no geometry source is selected, "Siguiente" is disabled and a hint explains that no geometry was found.

### US2.3 Detect coordinate columns in CSV
As Camila, I want the wizard to find my latitude and longitude columns, so that my survey points land on the map without manual mapping.
Priority: Must Have. Traces: FR2.2. Depends on: US2.1.
- AC2.3.1 Given a CSV with columns named lat and lon, latitud and longitud, or x and y, when step 2 opens, then the pair is pre-selected as the geometry source.
- AC2.3.2 Given a CSV where the detected columns hold values outside valid latitude or longitude ranges, when step 2 opens, then a warning names the count of offending rows.

### US2.4 Choose the geometry manually when nothing is detected
As Camila, I want to pick the geometry columns myself when detection fails, so that an unusual header does not block my upload.
Priority: Must Have. Traces: FR2.2. Depends on: US2.3.
- AC2.4.1 Given a CSV with no recognisable coordinate columns, when step 2 opens, then no geometry source is selected, "Siguiente" is disabled, and a hint asks me to choose the columns.
- AC2.4.2 Given that state, when I select two numeric columns as longitude and latitude, then "Siguiente" is enabled.

### US2.5 Name the layer and choose visible columns
As Andrés, I want to name the layer and choose which attributes are shown in the popup, so that internal identifiers stay out of the map.
Priority: Must Have. Traces: FR2.3. Depends on: US2.2, US2.3.
- AC2.5.1 Given step 2, when I untick a column, then it is absent from the popup after the load while still stored.
- AC2.5.2 Given step 1, when I leave the layer name empty, then "Siguiente" stays disabled with a message.

### US2.6 Confirm and run the load in the background
As Camila, I want to confirm a summary and let the load run in the background, so that I can keep using the map while it processes.
Priority: Must Have. Traces: FR2.6, FR6.3, NFR2. Depends on: US2.5.
- AC2.6.1 Given step 3, when I click "Cargar", then the wizard closes, the layer appears in the list as loading, and the status bar shows one running job.
- AC2.6.2 Given a running job, when it completes, then the layer state changes to active without a page reload.
- AC2.6.3 Given a job that fails entirely, when it ends, then the layer shows a warning state and "Ver detalle" opens the failure reason.
- AC2.6.4 Given a 10 MB CSV fixture, when the load runs on the shared environment, then the layer is active within 5 minutes.

### US2.7 See every row that failed to load
As Andrés, I want each failed row recorded with its reason, so that a load never drops data silently.
Priority: Must Have. Traces: FR2.7, FR6.3. Depends on: US2.6.
- AC2.7.1 Given a CSV with 1,000 rows of which 3 have invalid coordinates, when the load finishes, then 997 features exist, the job status reports 3 failed rows with their row numbers and reasons, and the layer list shows "3 filas con error".
- AC2.7.2 Given a load with failed rows, when I open "Ver detalle", then I can read every failed row's number and reason.

### US2.8 Keep the original file
As Andrés, I want the original uploaded file kept with the layer, so that a load can be audited or repeated.
Priority: Must Have. Traces: FR2.4. Depends on: US2.6.
- AC2.8.1 Given a completed load, when I inspect the layer's details, then the original file name, size and upload time are shown and the stored file matches the upload byte for byte.
- AC2.8.2 Given the stored file cannot be written, when the load starts, then the job fails before any feature is written and the reason names storage.

### US2.9 Convert coordinates to the map system
As Andrés, I want every loaded geometry converted to the map coordinate system, so that official data in the national projection lands in the right place.
Priority: Must Have. Traces: FR2.5. Depends on: US2.6, US9.1.
- AC2.9.1 Given a GeoJSON declaring EPSG:9377, when it loads, then the reference point from the fixture set appears within 1 metre of its known WGS84 position.
- AC2.9.2 Given a CSV with latitude and longitude, when it loads, then no conversion changes the values beyond floating-point tolerance.
- AC2.9.3 Given a file declaring an unsupported coordinate system, when it loads, then the job fails with a message naming the system.
- AC2.9.4 Given a reference point, when it is converted from EPSG:9377 to EPSG:4326 and back, then the round trip returns the original coordinates within 1 centimetre.

### US2.10 See the new layer on the map
As Camila, I want the map to zoom to my new layer when the load completes, so that I can see the result at once.
Priority: Must Have. Traces: FR2.8. Depends on: US2.6.
- AC2.10.1 Given a completed load, when the layer becomes active, then it is visible with a default style and the map extent contains all its features.
- AC2.10.2 Given a completed load of a layer with zero features, when it becomes active, then the map does not move and the list shows "0 elementos".

### US2.11 Cancel the wizard
As Laura, I want to leave the add-data wizard at any step, so that I am never stuck in it, including on a tablet.
Priority: Must Have. Traces: FR2.1. Depends on: US2.1.
- AC2.11.1 Given any wizard step, when I tap "Cancelar" or press Escape, then a confirmation asks whether to discard, and confirming closes the wizard with nothing uploaded.

## Group 3: Official cadastre (FR3)

### US3.1 Load cadastral parcels from an official-format file
As Andrés, I want a developer-run job that loads a cadastral parcel file in the official format and projection, so that the loading path is proven on a small fixture before the full city.
Priority: Must Have. Traces: FR3.1, FR2.5. Depends on: US2.9, US9.1.
- AC3.1.1 Given the pilot-localidad parcel fixture in EPSG:9377, when the job runs, then a "Predios catastrales" layer exists with one feature per parcel and the number of floors where published.
- AC3.1.2 Given the job, when it finishes, then the fixture's reference parcel centroid is within 1 metre of its known WGS84 position.
- AC3.1.3 Given a malformed input file, when the job runs, then it fails with a clear message and no partial layer is left active.

### US3.2 Inspect the cadastre like any layer
As Camila, I want to toggle, style and click the cadastre layer, so that I can compare my points against parcels.
Priority: Must Have. Traces: FR3.2. Depends on: US3.1, US4.3.
- AC3.2.1 Given the cadastre layer is active, when I click a parcel, then the popup shows its code, area, floors and localidad.

### US3.3 Reload the cadastre from a newer release
As Andrés, I want the cadastre job to be repeatable, so that a newer official release replaces the layer without manual database work.
Priority: Should Have. Traces: FR3.3. Depends on: US3.1.
- AC3.3.1 Given an existing cadastre layer, when the job runs again with a newer file, then the layer is replaced atomically and the old data is gone.
- AC3.3.2 Given the second run fails midway, when it ends, then the previous layer is still intact and active.

### US3.4 Load the full-city cadastre
As Andrés, I want the job run against the real IDECA release for all of Bogotá, so that the shared map carries the official base layer.
Priority: Must Have. Traces: FR3.1. Depends on: US3.1, US9.2.
- AC3.4.1 Given the downloaded IDECA parcel release, when the job runs, then the layer's feature count equals the release's record count and the job report lists any rows that failed.
- AC3.4.2 Given the full layer, when I view it at city scale, then parcels render and a click on a parcel in a known localidad returns its attributes.

## Group 4: Layer panel and map exploration (FR4)

### US4.1 Toggle layers
As Laura, I want to show and hide layers from the panel, so that I see only what I am comparing.
Priority: Must Have. Traces: FR4.1. Depends on: US1.1.
- AC4.1.1 Given an active layer, when I untick it, then it disappears from the map immediately and its checkbox state persists for the session.
- AC4.1.2 Given 25 layers, when the panel renders, then the list scrolls inside the panel and every layer name wraps rather than truncating.

### US4.2 Style a layer
As Laura, I want to change a layer's colour, opacity and point size, so that overlapping layers are distinguishable.
Priority: Must Have. Traces: FR4.2. Depends on: US4.1.
- AC4.2.1 Given a layer, when I change its colour, then the map updates within the same interaction without reload.
- AC4.2.2 Given a polygon layer, when I set opacity to 30%, then the base map is visible through it.
- AC4.2.3 Given a style change, when I reload the page, then the layer keeps the style I set.

### US4.3 Inspect a feature in a popup
As Camila, I want a popup with a feature's attributes where I click, so that I read values without leaving the map.
Priority: Must Have. Traces: FR4.3. Depends on: US4.1.
- AC4.3.1 Given a visible layer, when I click one of its features, then a popup anchored at the click lists the layer's visible attributes.
- AC4.3.2 Given an open popup, when I press Escape, click or tap elsewhere on the map, then it closes.
- AC4.3.3 Given attribute loading fails, when the popup opens, then it shows "No se pudieron obtener los atributos" with a "Reintentar" link.
- AC4.3.4 Given an attribute with no value, when the popup renders, then the cell shows "Sin valor".

### US4.4 See position, scale and jobs in the status bar
As Andrés, I want cursor coordinates, scale and running-job count at the bottom of the map, so that I keep orientation while loads run.
Priority: Must Have. Traces: FR4.4. Depends on: US1.1.
- AC4.4.1 Given the map, when I move the cursor, then the status bar shows its latitude and longitude to four decimals.
- AC4.4.2 Given two running jobs, when I look at the status bar, then it shows "Trabajos: 2 en curso".
- AC4.4.3 Given no running jobs, when I look at the status bar, then it shows "Trabajos: ninguno en curso".

### US4.5 Delete a layer
As Camila, I want to delete a layer I uploaded, so that mistakes do not clutter the shared map.
Priority: Must Have. Traces: FR4.5. Depends on: US2.10.
- AC4.5.1 Given a layer I uploaded, when I choose delete and confirm, then the layer, its features and its stored file are removed and it disappears for every user.
- AC4.5.2 Given the confirmation dialog, when I cancel, then nothing changes.

### US4.6 Set who can see a layer
As Andrés, I want to choose which users can see a layer I uploaded, so that an agency dataset stays within the people allowed to see it.
Priority: Must Have. Traces: FR4.6. Depends on: US2.10.
- AC4.6.1 Given a layer I uploaded, when I set its visibility to two named users, then those two see it in the layer list and the fourth user does not.
- AC4.6.2 Given a new upload, when I do not change visibility, then the layer is visible to all four users.
- AC4.6.3 Given a layer I did not upload, when I open its settings, then the visibility control is not offered.

### US4.7 Empty state
As Camila, I want guidance when I have no visible layers, so that a blank map does not look broken.
Priority: Should Have. Traces: FR4.7. Depends on: US1.1.
- AC4.7.1 Given no visible layers, when the panel renders, then it shows "Aún no hay capas. Agrega tus datos o carga el catastro oficial." with the "Agregar datos" action.

### US4.8 Collapse and expand the side panel
As Laura, I want to collapse the layer panel, so that the map takes the full width during a demo.
Priority: Must Have. Traces: FR4.1. Depends on: US4.1.
- AC4.8.1 Given the panel is open, when I click or tap its collapse control, then the map fills the width and the control remains visible to expand it again.

### US4.9 Enforce visibility in tiles and popups
As Andrés, I want a private layer's data to be unreachable by users outside its allowed set, even by direct request, so that visibility is a real control and not a display filter.
Priority: Must Have. Traces: FR4.6, FR1.4. Depends on: US4.6.
- AC4.9.1 Given a private layer, when a non-allowed signed-in user requests its tiles or attributes directly, then the response is an authorization error and no feature data is returned.
- AC4.9.2 Given a private layer, when an allowed user requests its tiles, then they are served.
Note: the enforcement mechanism (database-level policy or a filter in front of the tile server) is a design decision for Domain or Contract Design; the criteria hold for either.

## Group 5: 3D view of the pilot localidad (FR5)

### US5.1 Switch between 2D and 3D
As Laura, I want an explicit 2D / 3D toggle that keeps my position, so that I can move between views without losing my place.
Priority: Must Have. Traces: FR5.1. Depends on: US4.1.
- AC5.1.1 Given the 2D view, when I click "3D", then the 3D view opens centred on the same location and the toggle shows 3D selected.
- AC5.1.2 Given the 3D view, when I click "2D", then the flat map returns at the same location.
- AC5.1.3 Given the 3D view is loading, when terrain tiles are still arriving, then the banner "Cargando terreno..." is shown until they finish.

### US5.2 See terrain in the pilot localidad
As Laura, I want real terrain relief inside the pilot localidad, so that the ground shape is visible.
Priority: Must Have. Traces: FR5.2. Depends on: US5.1, US9.2.
- AC5.2.1 Given the 3D view over the pilot localidad, when terrain loads, then the fixture reference point's elevation is within 2 metres of the terrain-model value.

### US5.3 Understand the pilot boundary
As Laura, I want the 3D view to tell me when I am outside the pilot localidad, so that flat terrain is not mistaken for a bug.
Priority: Must Have. Traces: FR5.3. Depends on: US5.2.
- AC5.3.1 Given the 3D view outside the pilot localidad, when it renders, then terrain is flat, no buildings appear, and a banner explains the pilot boundary.

### US5.4 Keep my layers in 3D
As Camila, I want my uploaded layers and the cadastre visible and clickable in 3D, so that I can read attributes against the relief.
Priority: Must Have. Traces: FR5.4. Depends on: US5.1, US4.3.
- AC5.4.1 Given a visible point layer, when I switch to 3D, then its points are drawn on the terrain and a click opens the same popup as in 2D.
- AC5.4.2 Given a layer hidden in 2D, when I switch to 3D, then it stays hidden.

### US5.5 Fall back when terrain is unavailable
As Laura, I want the view to return to 2D with a message when terrain cannot load, so that I am never stuck on a broken globe.
Priority: Should Have. Traces: FR5.5. Depends on: US5.1.
- AC5.5.1 Given terrain tiles fail to load, when I switch to 3D, then the view returns to 2D and shows "La vista 3D no está disponible ahora".

### US5.6 Compute building heights per parcel
As Andrés, I want each parcel's building height computed as surface model minus terrain model, so that the 3D buildings have real heights.
Priority: Must Have. Traces: FR5.2. Depends on: US3.1, US9.2.
- AC5.6.1 Given the pilot terrain and surface fixtures, when the height job runs, then the fixture's reference parcel gets a height within 1 metre of its known value stored on the parcel.
- AC5.6.2 Given a parcel outside the raster coverage, when the job runs, then its height is empty and the job report counts it, not silently zero.

### US5.7 See buildings extruded in 3D
As Laura, I want parcels inside the pilot localidad drawn as blocks of their real height, so that building mass is visible against the terrain.
Priority: Must Have. Traces: FR5.2. Depends on: US5.2, US5.6.
- AC5.7.1 Given the 3D view over the pilot localidad, when buildings render, then the reference parcel's extrusion height is within 1 metre of its stored height.
- AC5.7.2 Given a parcel with an empty height, when buildings render, then it is drawn flat, not as a default block.

## Group 6: Operational visibility (FR6)

### US6.1 Health endpoint
As Andrés, I want a health endpoint that reports database and storage availability, so that a broken deployment is obvious before a demo.
Priority: Must Have. Traces: FR6.1, NFR7. Depends on: none.
- AC6.1.1 Given a healthy deployment, when the endpoint is called, then it returns a success status naming database and storage as available.
- AC6.1.2 Given the database is unreachable, when the endpoint is called, then it returns a failure status naming the database.

### US6.2 Structured error logs
As Andrés, I want errors logged with timestamp, component and job or request identifier, so that a failed load can be traced.
Priority: Must Have. Traces: FR6.2, NFR7. Depends on: none.
- AC6.2.1 Given a load job fails, when I read the logs, then one structured entry names the job identifier, the component and the reason.

### US6.3 Measure map tile responsiveness
As Andrés, I want the cadastre tile response time measured and reported, so that a latency target can be set after the proof of concept.
Priority: Should Have. Traces: NFR1. Depends on: US3.4.
- AC6.3.1 Given the full cadastre layer, when a scripted run requests city-scale tiles as 4 concurrent users, then a report records the median and 95th-percentile response time.

## Group 7: Interface (FR7)

### US7.1 Spanish interface
As Camila, I want every label and message in Spanish, so that the team reads the tool in its working language.
Priority: Must Have. Traces: FR7.1. Depends on: none.
- AC7.1.1 Given any screen, when it renders, then all labels, buttons and messages are in Spanish.

### US7.2 Tablet use
As Laura, I want to pan, zoom and use the layer panel by touch on a tablet, so that I can drive a demo without a mouse.
Priority: Should Have. Traces: FR7.2. Depends on: US4.1, US5.1.
- AC7.2.1 Given a tablet viewport, when I use touch gestures, then the map pans and zooms, the panel collapses and expands, and controls are at least 44 by 44 pixels.
- AC7.2.2 Given the 3D view on a tablet, when I drag with two fingers, then the view tilts.

## Group 8: Later capabilities (title level, no acceptance criteria)

- US8.1 Geocode uploaded rows by address. Priority: Should Have. Traces: requirements Out of Scope (PU-06).
- US8.2 Place uploaded rows by reference codes (UPZ, localidad, barrio, manzana). Priority: Should Have. Traces: PU-06.
- US8.3 Pull a dataset from an external service by configuration. Priority: Could Have. Traces: PU-08.
- US8.4 Run a flood-risk scenario and see its result layer. Priority: Could Have. Traces: PU-09.

## Group 9: Enablers

### US9.1 Reference fixtures for spatial tests
As a developer, I want a small versioned fixture set with known coordinates in both projections, a reference parcel and a reference elevation, so that the numeric acceptance criteria are runnable tests.
Priority: Must Have. Traces: NFR6, C-C. Depends on: none.
- AC9.1.1 Given the repository, when the test suite starts, then it loads the fixture set into the disposable PostGIS container and every numeric criterion (AC2.9.1, AC2.9.4, AC3.1.2, AC5.2.1, AC5.6.1, AC5.7.1) references a fixture value.

### US9.2 Choose the pilot localidad and obtain its data
As Andrés, I want the pilot localidad chosen and its parcel, terrain and surface products downloaded and verified, so that the cadastre and 3D stories have real inputs.
Priority: Must Have. Traces: C-D, requirements Open Questions. Depends on: none.
- AC9.2.1 Given the chosen localidad, when its products are downloaded, then each file's coordinate system is confirmed as EPSG:9377 (or recorded otherwise) and the files are stored with their source and download date.

## Story Map Summary

| Group | Stories | Must | Should | Could |
|---|---|---|---|---|
| 1 Accounts | 4 | 4 | 0 | 0 |
| 2 Upload | 11 | 11 | 0 | 0 |
| 3 Cadastre | 4 | 3 | 1 | 0 |
| 4 Layer panel | 9 | 8 | 1 | 0 |
| 5 3D | 7 | 6 | 1 | 0 |
| 6 Operations | 3 | 2 | 1 | 0 |
| 7 Interface | 2 | 1 | 1 | 0 |
| 8 Later | 4 | 0 | 2 | 2 |
| 9 Enablers | 2 | 2 | 0 | 0 |
| Total | 46 | 37 | 7 | 2 |

## Assumptions & Open Questions

- Default visibility for a new layer is "all users" (AC4.6.2); the requirements listed this as open and the stories adopt the simpler default. [assumption]
- Session lifetime is 12 hours (AC1.2.1, AC1.2.2); a design choice adopted so the criterion is testable. [assumption]
- The visibility enforcement mechanism for tiles (US4.9) is decided at design; both candidate mechanisms satisfy the criteria. [assumption]
