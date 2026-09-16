**Collaborator:** aidlc-developer-agent

## Contribution

Scope of this review: naming conventions, layer/module boundaries for the
modular monolith (`catalog`, `ingestion`, `models`), error handling at
integration boundaries, file/repository organization (Docker Compose
layout, single repo vs split), and code-style/formatter/linter conventions
for Python and TypeScript/Angular. Assessed against
`project-description.json` (architecture doc v0.1) and
`ideation/scope-definition/scope-document.md`. Project is confirmed
GREENFIELD — no code, CI, or linter config exists yet, so every finding
below is a recommendation to put to the interview, not an observation of
existing code.

### 1. Naming conventions

- The architecture document's own SQL schema (§4) already establishes a
  convention worth making explicit team policy rather than leaving implicit:
  plural snake_case table names (`datasources`, `layers`, `features`,
  `scenarios`), singular snake_case columns, and `_id`-suffixed foreign
  keys. Recommend stating this as the DB naming rule rather than relying on
  the schema example to carry it silently.
- Python: `snake_case` for modules/functions/variables, `PascalCase` for
  Pydantic models and SQLAlchemy models — standard, no objection to the lead
  draft's org-default carry-forward.
- TypeScript/Angular: `camelCase` for variables/functions, `PascalCase` for
  classes/components/interfaces, kebab-case for component selectors and file
  names (`layer-panel.component.ts`) — Angular CLI's own generator default;
  worth stating explicitly since Angular has a stronger house style than
  bare "language idiomatic" implies.
- Not yet covered anywhere in the draft: a naming rule for the `models/`
  analysis module's job/model identifiers (the schema uses a free-text
  `model` column, e.g. `"flood_v1"`). Recommend a `snake_case` + version-suffix
  convention (`flood_v1`, `flood_v2`) fixed now, before more than one model
  exists, since renaming a `scenarios.model` value later is a live-data
  migration, not a refactor.

### 2. Layer and module boundaries (modular monolith)

The architecture document names `catalog`, `ingestion`, and `models` as the
module seams (§2, principle 1) but does not specify the boundary rule
between them. This is the single most consequential omission for a 1-2
person team, because without an explicit rule the "seams to extract
services from later" claim in §2 erodes silently as soon as two modules
share a database session. Recommend affirming, as team practice:

- Each module (`catalog`, `ingestion`, `models`) owns a Python package under
  `backend/app/` with its own routers/schemas/service layer. A module may
  read another module's data only through that module's service functions,
  never by importing another module's SQLAlchemy models or issuing direct
  queries against tables it doesn't own. Concretely: `ingestion` writing a
  `features`/`layers` row directly is *not* fine unless `catalog` exposes a
  `create_features()`/`create_layer()` service call for it to use; `models`
  reading `layers`/`features` to run a simulation must go through
  `catalog`'s read service, not a raw query.
- This mirrors the `MapAdapter` isolation principle the architecture
  document already applies to the frontend (§2, principle 4) — the same
  discipline is missing on the backend side of the same document.
- Without this rule, the stated goal of §2 ("module boundaries are the
  seams along which services can be extracted later") is aspirational only;
  the interview should confirm the team wants to hold this line from the
  first Bolt, since retrofitting it after `ingestion` and `models` have
  cross-imported each other's models is expensive for a 1-2 person team.

### 3. Error handling at integration boundaries

`phases/construction.md` (guardrail, mandatory for this project) requires
error handling at API calls, database operations, and file I/O, surfaced or
logged rather than silently swallowed, with recoverable/fatal distinguished.
The lead draft's `discovered-rules.md` does not yet have a candidate rule
for this, and the architecture document names the boundaries but not the
handling policy. Three boundaries need an explicit rule, each already named
in the architecture but left open:

- **File parsing (ingestion, §5 steps 2-4).** Format detection, schema
  inference, and geometry detection can each fail per-record or per-file.
  §5 step 7 already commits to "report job status and validation warnings
  (unmatched codes, failed geocodes) back to the API" — recommend this
  become a stated Mandated rule (`ALWAYS report per-record ingestion
  failures on the job/datasource record; never drop a failed row silently`)
  rather than remaining descriptive prose in the architecture doc only.
  Suggest a job-status enum that distinguishes `failed` (fatal, e.g.
  unreadable file) from `succeeded_with_warnings` (partial: N rows loaded,
  M rows failed geocoding/reference-code match) — the current schema's
  `status: text` column has no defined value set yet.
- **External services (geocoding, WFS/REST connector, §5 step 4 and §9).**
  No timeout, retry, or backoff policy is stated for the Google Geocoding
  API calls or the configuration-driven external API/WFS connector. Given
  a 1-2 person team and no ops function, recommend a bounded-retry default
  (fixed small retry count with backoff, then fail the record/job rather
  than hang) be affirmed now rather than discovered under a live outage
  later.
- **Database (reprojection + PostGIS writes).** C-02 (silent CRS
  misplacement) is already flagged by the lead draft for a testing floor;
  it is equally an error-handling boundary — a reprojection that raises
  (unrecognized source CRS, out-of-bounds coordinates after transform)
  must fail the record loudly, not write untransformed or null geometry.
  Recommend pairing the lead draft's proposed reprojection test with an
  explicit rule that a failed reprojection is a per-record ingestion
  failure (per the job-status point above), never a silent pass-through.
- **API boundary (FastAPI → Angular).** Not yet addressed anywhere in the
  draft: a convention for mapping domain/service exceptions to HTTP error
  responses (a central FastAPI exception handler with a consistent error
  body shape) rather than ad hoc `HTTPException` raises scattered per
  route. Small addition, but worth deciding once rather than per endpoint.

### 4. File and repository organization

Not addressed in the lead draft or evidence.md, and there is no
`project.md` § Tech Stack entry yet to defer to. Recommend the interview
resolve:

- **Single repo vs split.** Given one shared Docker Compose file already
  named in the architecture (§9) and a 1-2 person team, recommend a single
  repository with top-level `backend/` (Python/FastAPI + ingestion +
  models), `frontend/` (Angular + CesiumJS), and `infra/` (Docker Compose,
  and later IaC) directories, over a multi-repo split. A split adds
  cross-repo versioning overhead this team size can't absorb, and nothing
  in the architecture document argues for independent deployability yet.
- **Docker Compose service layout.** §9 names Cloud Run services (API,
  Martin, TiTiler, frontend) and Cloud Run Jobs (ingestion/model jobs) but
  the local Compose equivalent isn't enumerated. Open question the compose
  file itself will force an answer to: §5's "Cloud Run Jobs, or Celery/RQ
  locally" is an unresolved either/or — if Celery/RQ is chosen for local
  dev, Compose needs a broker service (Redis) and a worker service, which
  the Cloud Run Jobs path doesn't need at all. This should be decided
  before the first Compose file is written, not discovered mid-ingestion
  work.
- **Migrations.** No tool is named for PostGIS/SQL schema migrations
  (Alembic is the standard pairing with SQLAlchemy/FastAPI). The lead
  draft flags PostGIS/SQL convention as "fully open" — recommend narrowing
  that specifically to "which migration tool", since file-naming
  convention is a consequence of the tool choice, not an independent
  question.

### 5. Code-style / formatter / linter

Agree with the lead draft's Black/Ruff (Python) and Prettier/ESLint
(Angular) baseline — it matches `org.md`'s language-default policy and the
confirmed stack. Additions the draft doesn't cover:

- **Python type checking.** FastAPI's value proposition (request/response
  validation, OpenAPI generation) depends on consistent type hints via
  Pydantic. Recommend adding `mypy` (or Ruff's own type-aware rules, if the
  team prefers one tool) to the interview options — this is a code-quality
  gap the org-default formatter/linter pairing doesn't cover, and
  `org.md`'s Code Style section explicitly says an agent's own suggestion
  fires only when the linter config doesn't already cover it. Ruff alone
  does not perform type checking.
- **Angular strict mode.** Recommend confirming Angular's `strict: true`
  compiler flag (strict template type checking, strict null checks) at
  project scaffold time — retrofitting strict mode after component code
  exists is materially more expensive than starting with it, and the
  architecture doc doesn't record a decision either way.
- **Pre-commit enforcement.** Not addressed: whether Black/Ruff/Prettier/
  ESLint run via a pre-commit hook (fast, local) versus CI-only (per
  `org.md`'s "linter runs in CI, failure blocks the PR"). For a 1-2 person
  team without a PR-review gate necessarily in place (open question raised
  in Way of Working), a pre-commit hook is a cheaper enforcement point than
  relying on CI alone. Flag as an interview option, not a default.

## Positions

- AGREE: `team-practices.md`'s Testing Posture section — test-after as
  default with a targeted reprojection/geometry-detection regression floor
  is well-reasoned given C-02 and C-04, and correctly separates that floor
  from the scope-dependent coverage question.
- AGREE: `discovered-rules.md`'s public-data-only Forbidden rule and its
  documented resolution of the C-05 contradiction — the scope document's
  newer confirmed answer is correctly treated as authoritative, and the
  evidence.md write-up of that resolution is clear enough for the human
  gate to confirm without re-deriving it.
- OBJECT: `discovered-rules.md` has no Mandated candidate for ingestion
  error surfacing (§5 step 7's "report job status and validation warnings"
  and `phases/construction.md`'s error-handling guardrail), even though it
  already has one for reprojection (CRS) and one for raw-file provenance
  that come from the same architecture section. This is a gap, not a
  disagreement with what's there — recommend the lead add a Mandated rule
  along the lines of "ALWAYS record per-record ingestion failures on the
  job/datasource status rather than dropping the row silently" at
  integration, sourced from §5 step 7 and the construction-phase guardrail
  (see §3 of my Contribution above for the full rationale).
- OBJECT: `team-practices.md`'s Code Style section marks "SQL/PostGIS
  migrations: no team convention yet — open question" without narrowing it
  to a concrete decision the interview can resolve in one question. The
  open question is specifically "which migration tool" (Alembic is the
  default pairing with the confirmed SQLAlchemy/FastAPI stack) — narrowing
  it will make the interview question answerable rather than open-ended
  (see §4 of my Contribution above).
