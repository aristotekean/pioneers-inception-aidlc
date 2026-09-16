# Requirements Analysis Questions

Context carried forward: proof of concept for four researchers; must-haves are self-serve CSV/JSON upload with geometry detection, a layer panel (toggle, inspect, basic styling), official cadastre on the map, and a 3D view of the pilot localidad; per-user local login is now a must-have (practices interview Q14, superseding the earlier no-login decision); Spanish UI, desktop and tablet; TDD with an 80% coverage floor; GitHub Actions; one Cloud Run environment; budget under USD 100 per month. The questions below fill the gaps needed to write testable requirements.

## Q1. How are the four researchers' accounts created and how do they sign in?

A. An administrator among the four creates accounts; sign-in with email and password
B. Researchers register themselves with an invitation code; sign-in with email and password
C. Sign-in through institutional Google accounts; no local passwords
D. Not yet defined
X. Other (please specify)

[Answer]: X. Other — all will be same role admin (account creation and sign-in mechanism clarified in Q8)

## Q2. What roles exist in the proof of concept?

A. One role: every researcher can upload, style, delete their own layers and set their visibility
B. Two roles: an administrator who manages accounts, plus researchers
C. Two roles: an administrator who also owns every layer's visibility, plus researchers who only upload
D. Not yet defined
X. Other (please specify)

[Answer]: A. One role: every researcher can upload, style, delete their own layers and set their visibility

## Q3. Which file formats must the self-serve upload accept?

Your document names CSV and JSON first. The official cadastre is usually published as shapefiles or GeoPackage; those could be loaded by a developer-run job rather than the upload screen.

A. CSV and JSON/GeoJSON only; official datasets are loaded by a developer-run job
B. CSV, JSON/GeoJSON, zipped shapefile and GeoPackage through the upload screen
C. CSV and GeoJSON only, nothing else
D. Not yet defined
X. Other (please specify)

[Answer]: A. CSV and JSON/GeoJSON only; official datasets are loaded by a developer-run job

## Q4. What is the largest file a researcher may upload through the screen?

This bounds validation, job duration and the budget.

A. Up to 10 MB, roughly tens of thousands of rows
B. Up to 100 MB, roughly hundreds of thousands of rows
C. No fixed limit for the proof of concept
D. Not yet defined
X. Other (please specify)

[Answer]: A. Up to 10 MB, roughly tens of thousands of rows

## Q5. How responsive must the map be with the full cadastre loaded?

A measurable target lets the build stage verify it. Four concurrent users is the reference load.

A. Map tiles for the cadastre layer render within 1 second at city scale for 4 concurrent users
B. Within 3 seconds is acceptable for the proof of concept
C. Best effort, no numeric target
D. Not yet defined
X. Other (please specify)

[Answer]: C. Best effort, no numeric target

## Q6. What operational visibility does the proof of concept need?

A. A health endpoint, structured error logs, and per-job status visible in the interface
B. Error logs only; job status visible in the interface
C. Nothing beyond what the cloud platform shows by default
D. Not yet defined
X. Other (please specify)

[Answer]: A. A health endpoint, structured error logs, and per-job status visible in the interface

## Q7. Can a researcher remove a layer they uploaded?

A. Yes: the uploader (or an administrator) can delete a layer and its data
B. Layers can be hidden but never deleted in the proof of concept
C. Not yet defined
X. Other (please specify)

[Answer]: A. Yes: the uploader (or an administrator) can delete a layer and its data

## Q8. Follow-up: with everyone in the same administrator role, how are the four accounts created and how do they sign in?

A. A developer seeds the four accounts at deployment; sign-in with email and password
B. Any signed-in researcher can create another account from the interface; sign-in with email and password
C. Sign-in through institutional Google accounts; the four allowed emails are configured at deployment
D. Not yet defined
X. Other (please specify)

[Answer]: A. A developer seeds the four accounts at deployment; sign-in with email and password

## Consolidated Summary Confirmation

- Accounts: one role with administrator rights for all four researchers; accounts seeded by a developer at deployment; sign-in with email and password
- Every researcher can upload, style, delete their own layers and set each layer's visibility
- Upload accepts CSV and JSON/GeoJSON up to 10 MB; official datasets are loaded by a developer-run job
- Map responsiveness: best effort, no pass/fail latency threshold; tile response time is measured and reported
- Operations: a health endpoint, structured error logs, per-job status in the interface
- Layers can be deleted by their uploader or any administrator

Does this all look correct before I generate the requirements artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
