# Evidence: Bogotá Urban Digital Twin — Practices Discovery

Status: **final, at integration (Step 5)**. Extends the Step 2 lead draft
with what each support agent inspected, the interview's per-question
decisions, the superseding authentication decision, and remaining
uncertainty.

## What Each Participant Inspected or Inferred

- **Lead (aidlc-pipeline-deploy-agent, Step 2 draft)**: the stage
  definition's greenfield branch; `org.md`'s five matching sections as
  suggested defaults; `team.md` (confirmed empty — first run, not a re-run);
  `project.md` (empty aside from a prior UI-language correction unrelated to
  this stage); the project's `project-description.json` architecture
  document in full (system overview, component table, data model, ingestion
  pipeline, visualization strategy, 3D data pipeline, GCP deployment table
  §9, roadmap); `ideation/scope-definition/scope-document.md`;
  `ideation/feasibility/constraint-register.md` (all four constraint tables
  plus its open questions); and a directory listing confirming no prior
  draft existed. Repository root confirmed GREENFIELD — no application
  code, CI configuration, or linter config exists yet.
- **aidlc-quality-agent**: reviewed the lead draft's Testing Posture section
  and proposed concrete coverage tooling (`pytest`/`pytest-cov`,
  `testcontainers-python` with the `postgis/postgis` image, Karma/Jest for
  Angular), a per-module coverage-floor split, and the detailed
  spatial-specific test design (reprojection reference-point and round-trip
  tests, one test per geometry-detection path plus a negative case, tile-
  output structural tests) later carried into `team-practices.md`.
- **aidlc-developer-agent**: reviewed module/layer boundaries, naming, error
  handling, and file organization against the architecture document and
  scope document. Proposed the backend module-boundary rule (`catalog`,
  `ingestion`, `models` communicate only through each other's service
  layers, never by cross-module table access), narrowed the "PostGIS/SQL
  convention" open question to "which migration tool" (recommending
  Alembic), and flagged the missing Mandated rule for per-record ingestion
  failure reporting.
- **aidlc-devsecops-agent**: reviewed lint/format enforcement, SAST,
  secret/dependency/container scanning, and GCP-specific controls. Proposed
  concrete secret-handling mechanics (GCP Secret Manager for cloud
  credentials, `.env.example` with a gitignored `.env` locally) and raised
  the ingestion/upload authentication gap that became Q12/Q14 of the
  interview.

## Interview Decisions (practices-discovery-questions.md, all confirmed)

| # | Topic | Decision |
|---|-------|----------|
| Q1 | Branching/merge | Trunk-based, short-lived branches, squash-merged into `main` |
| Q2 | Review step | Self-review checklist, no second approver |
| Q3 | Walking skeleton | Yes — solo, built and approved first |
| Q4 | Test timing | Test-first everywhere (TDD) — stricter than the quality agent's proposed `custom` (test-first only for reprojection/geometry-detection); the human's broader choice is authoritative and is recorded as `Methodology: tdd` |
| Q5 | Coverage floor | Keep the `mvp` scope's 80% line-coverage floor plus CI, not raised further |
| Q6 | Deployment topology | One shared cloud environment, deployed on every merge to `main`, no staging |
| Q7 | Formatters/linters | Black+Ruff (Python), Prettier+ESLint (Angular/TS), `gofmt` if Go; enforced in CI, failing the merge |
| Q8 | Hard-rule affirmation | Only option C selected (no hardcoded credentials); options A (reprojection helper), B (raw-file provenance), D (no private data — superseded, see below), and E (no full-city rasters without a cost decision) were **not** affirmed as hard rules and remain ordinary practices/scope notes, not `## Mandated`/`## Forbidden` entries |
| Q9 | Ingestion failure handling | Hard rule affirmed: always record per-row load failures/warnings on job status, never drop a row silently |
| Q10 | CI platform | GitHub Actions |
| Q11 | Test database | Disposable PostGIS container per test run (testcontainers-style), small versioned fixtures |
| Q12 | Upload-path protection | "Other" — human indicated intent to add stronger auth than network-level-only protection, resolved by Q14 |
| Q13 | Migrations | Alembic, files named by timestamp and purpose |
| Q14 | Auth shape (follow-up to Q12) | Per-user login for the four researchers, managed locally — this reopens and supersedes the earlier scope-definition decision (see below) |
| Summary | Consolidated confirmation | "Looks correct" |

Note on Q8: because options A, B, D, and E were not selected, the
corresponding proposed rules from the lead's Step 2 `discovered-rules.md`
draft (reprojection-helper mandate, raw-file-provenance mandate,
private-data-forbidden, full-city-raster-forbidden) are **not** carried
into the affirmed `discovered-rules.md`. The reprojection and pilot-
localidad-scoping intents survive as ordinary team practice inside
`team-practices.md`'s Testing Posture and this project's existing scope
documents; they are simply not elevated to permanent hard-rule status by
this interview.

## Superseding Decision: Per-User Login Reopens the Private-Data Scope Decision

`scope-document.md` (Scope Decisions and Their Effects) had confirmed "the
proof of concept ingests public datasets only, so no identity or permission
model is required for the demo." The devsecops agent's contribution (Q12
open point) surfaced that this framing covered the read/view path but never
addressed whether the unauthenticated upload/ingestion path was also
exposed to the open internet — a materially different risk (budget
exhaustion, malicious file uploads) independent of whether the underlying
data is public.

At Q12, the human chose "Other — I will add better auth" rather than either
offered option (network-level-only protection, or accept the
unauthenticated-upload risk). Q14 resolved the exact shape: **per-user login
for the four researchers, managed locally** (option A), which the question
itself named as reopening the scope decision.

Downstream effect, recorded here per the project's own correction rule
("when a later confirmed answer removes a need recorded by an earlier
stage... record the effect in the current stage's artifact and leave the
earlier stage's artifacts untouched"):

- **Identity becomes a must-have** for this proof of concept, not deferred
  post-launch as `scope-document.md` originally framed it.
- **Private datasets become conditional**, not categorically out of scope:
  Q14's option A explicitly states "private datasets may be admitted once
  per-layer visibility exists." Whether any private dataset is actually
  ingested during this proof of concept remains open and is a
  requirements-analysis question, not decided here.
- `scope-document.md`, `constraint-register.md`, and every other
  ideation-phase artifact are left untouched, per the same correction rule.
  This evidence file is the record of the reopening; requirements-analysis
  is where the identity/permission model itself gets specified.

## Unresolved Uncertainty

- **GCP account not confirmed.** The architecture document and this
  interview both treat GCP/Cloud Run/Cloud SQL as the preferred deployment
  target (constraint-register C-10), but no artifact in this workflow
  confirms an actual GCP account or billing setup exists yet. The Deployment
  section of `team-practices.md` is written against this preference; if the
  account does not materialize, the deployment target itself — not just its
  environment topology — would need to be revisited.
- **Team's GIS/3D learning curve.** Constraint-register C-04 records no
  prior GIS/3D experience on the team. The TDD-everywhere decision (Q4) and
  the spatial-specific test design mitigate the *correctness* risk this
  creates (silent CRS misplacement, geometry-detection errors), but the
  *delivery-velocity* risk of a small team learning PostGIS, CesiumJS, and
  the raster/3D pipeline simultaneously is not something a testing-posture
  decision resolves, and remains a live risk for delivery-planning to
  account for in sequencing and estimate confidence.
- **Repository host** (GitHub, GitLab, other) was not asked directly as its
  own question, though Q10's GitHub Actions answer strongly implies GitHub
  as the host; devsecops's proposed supplementary controls (Dependabot,
  native secret scanning) that depend on the host being GitHub specifically
  are not separately confirmed and are left as an implementation-time
  choice rather than a discovered team practice.
