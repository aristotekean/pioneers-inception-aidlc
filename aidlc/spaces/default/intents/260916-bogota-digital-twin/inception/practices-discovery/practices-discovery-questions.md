# Practices Discovery Questions

Greenfield interview. The suggested answers come from the framework defaults specialised to this project by the release engineer's draft (`team-practices.md`, `discovered-rules.md`). Confirmed context: proof of concept, one or two developers full-time, scope `mvp` (which adds an 80% line-coverage floor and CI execution before merge), Python/FastAPI and Angular with CesiumJS, Docker Compose locally, GCP preferred but not yet confirmed, budget under USD 100 per month, public data only, pilot localidad for rasters.

## Q1. How do you want to manage branches and merges?

A. Trunk-based: short-lived feature branches, squash-merged into main so each piece of work is one commit (suggested default)
B. Trunk-based with direct commits to main and no pull requests, given one or two developers
C. A long-lived development branch with releases merged to main (GitFlow style)
D. Not yet defined
X. Other (please specify)

[Answer]: A. Trunk-based: short-lived feature branches, squash-merged into main so each piece of work is one commit

## Q2. What review step happens before a merge?

A. Self-review against a short checklist, then merge without a second approver
B. Pair review when both developers are available, self-review otherwise
C. A second approver is always required
D. Not yet defined
X. Other (please specify)

[Answer]: A. Self-review against a short checklist, then merge without a second approver

## Q3. Build a thin end-to-end slice first? A walking skeleton is a minimal version that runs the whole way through, built first to prove the pieces connect before the real features go in.

The scope document already names "one uploaded dataset visible on the map" as the first demo milestone, which is a walking skeleton in substance.

A. Yes: the first piece of work is that thin slice, built alone and approved before the rest starts (suggested)
B. No: the first piece of work runs like any other
C. Not yet defined
X. Other (please specify)

[Answer]: A. Yes: the first piece of work is that thin slice, built alone and approved before the rest starts

## Q4. When are tests written relative to the code?

A. Test-after: build each layer, then write and run its tests before moving on (suggested default)
B. Test-first everywhere (TDD)
C. Custom: test-first for coordinate conversion and geometry detection, where a silent error is costly; test-after elsewhere
D. Behaviour scenarios written first for each feature, unit tests after (BDD)
E. Not yet defined
X. Other (please specify)

[Answer]: B. Test-first everywhere (TDD)

## Q5. The mvp scope adds an 80% line-coverage floor and CI before merge. Keep it as is, or commit to something stricter?

Floors can be raised at this point, not lowered.

A. Keep the 80% floor and CI before merge as is
B. Stricter: 80% overall plus full coverage of the coordinate-conversion and geometry-detection code
C. Stricter: 90% overall
D. Not yet defined
X. Other (please specify)

[Answer]: A. Keep the 80% floor and CI before merge as is

## Q6. How and where does the proof of concept get deployed?

A. One shared environment on the cloud, deployed automatically on every merge to main; no separate staging (suggested for a proof of concept)
B. Staging deployed on merge, plus a manually approved production environment (framework default)
C. Local Docker Compose only until the cloud account exists, then option A
D. Not yet defined
X. Other (please specify)

[Answer]: A. One shared environment on the cloud, deployed automatically on every merge to main; no separate staging

## Q7. Which formatters and linters, and where are they enforced?

A. Black and Ruff for Python, Prettier and ESLint for Angular/TypeScript, gofmt if Go appears; enforced in CI and failing the merge (suggested)
B. The same tools, enforced only by local pre-commit hooks
C. Not yet defined
X. Other (please specify)

[Answer]: A. Black and Ruff for Python, Prettier and ESLint for Angular/TypeScript, gofmt if Go appears; enforced in CI and failing the merge

## Q8. Which of the proposed hard rules do you affirm? (select all that apply)

Affirmed rules become permanent "ALWAYS" / "NEVER" guardrails for this project.

A. ALWAYS convert ingested geometry to a single map coordinate system (EPSG:4326) through one shared helper before storing it
B. ALWAYS keep the raw uploaded file for provenance before processing it
C. NEVER hardcode credentials, API keys or secrets in source code
D. NEVER ingest private or restricted datasets in this proof of concept
E. NEVER process full-city rasters in this proof of concept without a separate cost decision
X. Other (please specify)

[Answer]: C

## Q9. Should failed rows during a data load be recorded rather than silently dropped?

The developer flagged that the draft mandates coordinate conversion and provenance but says nothing about load failures, while your document's ingestion step 7 reports warnings back to the user.

A. Yes, as a hard rule: ALWAYS record per-row load failures and warnings on the job status; never drop a row silently
B. Yes, as a normal practice, not a hard rule
C. No; a failed load can simply fail as a whole
D. Not yet defined
X. Other (please specify)

[Answer]: A. Yes, as a hard rule: ALWAYS record per-row load failures and warnings on the job status; never drop a row silently

## Q10. Which CI platform runs the checks before merge?

A. GitHub Actions
B. GitLab CI
C. Google Cloud Build
D. Not yet defined
X. Other (please specify)

[Answer]: A. GitHub Actions

## Q11. How do tests get a spatial database to run against?

A. A disposable PostGIS container started per test run (testcontainers style), seeded with small versioned fixtures
B. A shared long-lived development database
C. Not yet defined
X. Other (please specify)

[Answer]: A. A disposable PostGIS container started per test run (testcontainers style), seeded with small versioned fixtures

## Q12. The proof of concept has no login. How is the deployed tool, including its upload path, protected from the open internet?

The security engineer noted that an unauthenticated upload endpoint reachable by anyone exposes the budget to abuse even when the data itself is public.

A. Network-level protection only: private access (VPN, IP allowlist, or a single shared access key), no per-user login
B. Publicly reachable and unauthenticated; accept the risk for the proof of concept
C. Not yet defined
X. Other (please specify)

[Answer]: X. Other — I will add better auth

## Q13. Which tool manages database schema migrations?

A. Alembic (the usual pairing with a Python/FastAPI backend), migration files named by timestamp and purpose
B. Plain SQL migration files applied by a small script
C. Not yet defined
X. Other (please specify)

[Answer]: A. Alembic (the usual pairing with a Python/FastAPI backend), migration files named by timestamp and purpose

## Q14. Follow-up: "better auth" for the proof of concept means what, exactly?

Scope Definition recorded "no login for the proof of concept" and "private datasets stay out until per-layer visibility exists". Your Q12 answer may reopen that. I need the exact shape so the requirements stage does not guess.

A. Per-user login for the four researchers in the proof of concept (this reopens the scope decision: identity becomes a must-have, and private datasets may be admitted once per-layer visibility exists)
B. One shared credential or access key for the four researchers, no per-user identity (keeps the scope decision; protects the upload path)
C. Per-user login through an external identity provider (institutional Google or similar accounts), no local user management
D. Not yet defined
X. Other (please specify)

[Answer]: A. Per-user login for the four researchers in the proof of concept (this reopens the scope decision: identity becomes a must-have, and private datasets may be admitted once per-layer visibility exists)

## Consolidated Summary Confirmation

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
