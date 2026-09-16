# Team Practices: Bogotá Urban Digital Twin

Status: **affirmed** at the practices-discovery interview on 2026-09-16
(`practices-discovery-questions.md`, all 14 questions answered, summary
confirmed "Looks correct"). These practices are ready for promotion to
`aidlc/spaces/default/memory/team.md` and `project.md`.

## Way of Working

- **Trunk-based development, squash-merged.** All work merges to `main` via
  short-lived feature branches; each Bolt becomes one commit on the trunk
  (affirmed, Q1). For Construction worktrees, the base branch and merge
  target are both `main`, consistent with `org.md`.
- **Self-review before merge, no second approver.** Each change is checked
  against a short self-review checklist (tests green, coverage floor met,
  lint/format clean, no secret committed) before merging; the team does not
  require a second human approver, given the one-or-two-developer team size
  (affirmed, Q2).

## Walking Skeleton

- **Yes — the walking skeleton runs first, solo and gated.** The first piece
  of work is the thin end-to-end slice already named as the project's first
  demo milestone (one uploaded dataset visible on the map); it is built
  alone and explicitly approved before the rest of the work starts, per
  `org.md`'s solo/gated Bolt-1 pattern (affirmed, Q3). Per the risk-first
  sequencing already recorded at scope-definition, the raster/3D pipeline for
  the pilot localidad is sequenced immediately after this walking skeleton
  rather than deferred to a later phase.
- After the walking skeleton ships, the orchestrator fires the standard
  ladder prompt to decide how the remaining Bolts run (continue autonomously
  vs. gate every Bolt); that choice is recorded separately in
  `aidlc-state.md` when it is made, not here.

## Testing Posture

- **Methodology**: tdd
- **Ordering**: Write the failing test before writing the implementation for
  every testable layer — ingestion parsing, coordinate reprojection,
  geometry detection, API endpoints, and map/layer-panel components — then
  implement until it passes, for every Bolt starting with the walking
  skeleton (affirmed, Q4: "Test-first everywhere (TDD)").
- **Coverage floor**: keep the `mvp` scope's 80% line-coverage floor as is,
  enforced in CI before merge; not raised further (affirmed, Q5).
- **Spatial database for tests**: a disposable PostGIS container started per
  test run (testcontainers-style), seeded with small, versioned fixtures
  checked into the repository — not a shared long-lived development database
  (affirmed, Q11).
- **Spatial-specific tests** (in addition to the general TDD/coverage floor,
  given the project's own risk register — silent coordinate-system
  misplacement and no prior team GIS/3D experience):
  - Coordinate reprojection (EPSG:9377 → EPSG:4326): at least one
    reference-point test per ingested dataset type, asserting a known input
    coordinate reprojects to the expected output within a fixed tolerance,
    plus one round-trip test (reproject forward, then inverse-reproject) to
    catch transform-direction and datum-mismatch bugs.
  - Geometry detection: one smoke test per detection path — native geometry,
    lat/lon columns, reference-code join, geocoded address — against a small
    fixture, plus one negative test: an upload matching none of the four
    paths must fail with a clear, surfaced error, never a silent partial
    import.
- **Migration testing**: every Alembic migration applies cleanly to a fresh
  PostGIS test database as part of the same disposable-container test run.

## Deployment

- **CI**: GitHub Actions runs the check suite (lint, tests, coverage floor)
  before every merge to `main` (affirmed, Q10).
- **Local development**: Docker Compose, as already named in the
  architecture document.
- **Cloud**: one shared environment on Cloud Run (API, Martin, TiTiler,
  frontend), Cloud Run Jobs (ingestion/model jobs), and Cloud SQL for
  PostgreSQL/PostGIS, deployed automatically on every merge to `main`; no
  separate staging environment for this proof of concept (affirmed, Q6).
  GCP as the target platform is the team's preference but the account itself
  is not yet confirmed — see `evidence.md` for this open point.
- **Secrets**: sourced from the platform's secret manager (GCP Secret
  Manager) for the database connection, Cloud SQL credentials, and the
  commercial geocoding API key; never a plain `.env` file or an inlined
  value committed to the repository.

## Code Style

- **Formatters/linters, CI-enforced**: Black and Ruff (Python), Prettier and
  ESLint (Angular/TypeScript), `gofmt` if a Go service is later added;
  enforced in CI, failing the merge on violation (affirmed, Q7).
- **Migrations**: Alembic, the standard pairing with the SQLAlchemy/FastAPI
  backend; migration files named by timestamp and purpose (affirmed, Q13).
- **Naming conventions**: language-idiomatic — `snake_case` for Python
  modules/functions/variables, `PascalCase` for Pydantic/SQLAlchemy models;
  `camelCase` for TypeScript variables/functions, `PascalCase` for
  Angular classes/components/interfaces, kebab-case for component selectors
  and file names. Database tables use plural snake_case names with singular
  snake_case columns and `_id`-suffixed foreign keys, matching the
  architecture document's own schema.
- **Module boundaries**: each backend module (`catalog`, `ingestion`,
  `models`) owns its own package with its own routers/schemas/service layer.
  A module reads or writes another module's data only through that module's
  service layer — never by importing another module's ORM models or issuing
  a direct query against a table it doesn't own — so the module seams stay
  real extraction points rather than eroding as soon as two modules share a
  database session.
- **Angular strict mode**: `strict: true` is enabled at project scaffold
  time (strict template type checking, strict null checks). Judged
  proportionate here because the team has already committed to test-first
  everywhere, and retrofitting strict mode after component code exists is
  materially more expensive than starting with it.
