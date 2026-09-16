**Collaborator:** aidlc-devsecops-agent

## Contribution

### Lint/format and static analysis as CI gates

- **Agree with the draft's tool choices** (Black/Ruff for Python, Prettier/ESLint
  for Angular/TypeScript), with one addition: these must run as **blocking CI
  gates**, not only local pre-commit checks. `org.md`'s Code Style section
  already says linter failure blocks the PR — the draft's "Open for the
  interview" framing under Code Style doesn't yet say CI enforcement is part
  of the ask, so I'm making that explicit.
- **Propose adding SAST as a CI gate**, not currently covered by the draft at
  all:
  - Python: **Bandit** (or **Semgrep** with its free community ruleset,
    which also covers the TypeScript side with one tool instead of two —
    worth deciding at the interview given the 1-2 developer capacity
    constraint, C-06).
  - TypeScript/Angular: `eslint-plugin-security` at minimum, or Semgrep as
    above.
  - Neither AWS CodeGuru Security nor SonarQube's paid tiers fit the
    confirmed budget ceiling (C-09, USD 100/month) or the GCP-preferred
    platform (C-10); Bandit/ESLint-security/Semgrep-community are all free
    and language-appropriate for this stack.

### Secret scanning and secrets handling

- **Propose a concrete mechanism** for the "no hardcoded credentials" rule
  the lead already drafted under `discovered-rules.md`'s Forbidden section —
  the draft states the rule but not how it's enforced:
  - Secret scanning: **gitleaks** or **detect-secrets** as a pre-commit hook
    and a CI gate (both free, both suit a small repo).
  - Local development: a committed `.env.example` template with no real
    values; the actual `.env` is gitignored. Docker Compose reads from
    `.env` via `env_file`, never inlines values in `docker-compose.yml`.
  - Cloud deployment: Cloud Run supports referencing **GCP Secret Manager**
    secrets directly as environment variables at the platform level — this
    should be the mechanism for the database connection string, the Cloud
    SQL credentials, and the commercial geocoding API key (D-03), not plain
    Cloud Run environment variables set from CI. Secret Manager's cost at
    this scale (a handful of secrets, low access volume) is negligible
    against the USD 100/month ceiling, so budget is not a reason to defer
    this.

### Dependency and container image scanning

- Not addressed in the draft. Propose, proportionate to a 1-2 developer
  proof of concept:
  - `pip-audit` (Python) and `npm audit` (TypeScript) run in CI against the
    lockfiles.
  - **Trivy** (free, open source) for container image scanning before any
    image is pushed to Artifact Registry / deployed to Cloud Run — covers
    both the OS-package and application-dependency layers in the images
    named in the architecture document (API, Martin, TiTiler, ingestion
    jobs).
  - If the repository host is GitHub, Dependabot alerts are free and should
    be enabled as a low-effort supplement; this is an open question below
    since the repo host isn't stated in any artifact I read.

### Supply-chain controls proportionate to team size

- Pin dependencies via lockfiles (`requirements.txt` with hashes or
  `poetry.lock`; `package-lock.json`) rather than floating version ranges,
  so `pip-audit`/`npm audit`/Trivy scan a reproducible dependency set.
- Avoid `:latest` base image tags in any Dockerfile; pin to a specific
  digest or version tag, and prefer slim/distroless bases to shrink the
  scanned surface.
- These are lightweight, no-added-cost practices — appropriate for a
  2-person team with no dedicated security function, rather than a formal
  SBOM/provenance-attestation pipeline, which would be disproportionate at
  this stage.

### Input validation on the ingestion path (gap not covered by the draft)

- The architecture document's ingestion pipeline accepts uploaded files
  (shapefiles, GeoJSON, KML/GML, CSV with lat/lon or reference codes) and
  the constraint register's C-02 already flags the CRS-conversion risk, but
  no artifact I read addresses **malicious or malformed input** on that same
  path:
  - Enforce a file-type allowlist and a size limit at the upload boundary.
  - GML/KML/XML-based formats are XXE (XML External Entity) attack vectors
    if parsed with a naive XML parser — the ingestion parser must disable
    external entity resolution.
  - Shapefile/zip uploads are a zip-bomb vector — decompressed-size limits
    are needed before parsing.
  - This is a `NEVER trust ingested file content without validating
    type, size, and safe-parsing configuration first` — proposed as a hard
    rule below given it's a system boundary per `phases/construction.md`'s
    Security guardrail, not merely a suggestion.

### Cloud/GCP-specific controls

- **Least-privilege service accounts**: the architecture names several
  Cloud Run services and Cloud Run Jobs (API, Martin, TiTiler, ingestion/
  model jobs). Each should run under its own minimally-scoped service
  account (e.g., the tile servers need read-only database access; only the
  ingestion job needs write access), not the default Compute Engine service
  account, which is broad by default.
- **Cloud SQL network exposure**: Cloud SQL for PostgreSQL/PostGIS should
  use a private IP with the Cloud SQL Auth Proxy (or Cloud Run's built-in
  Cloud SQL connector) rather than a public IP with an authorized-networks
  allowlist, which is easy to misconfigure and easy to forget to narrow.
- **Audit logging**: Cloud Audit Logs should be left on for the project
  (default-on for most GCP services at no extra cost) so upload/ingestion
  actions are attributable even though the read/view side has no identity
  model. This matters more, not less, given the interview gap noted below.

### Testing posture — security-relevant addition

- **Agree** with the lead's proposal to add a targeted regression floor for
  CRS reprojection (C-02) and geometry-detection paths regardless of scope
  classification. **Addition**: those tests should include at least one
  malformed/malicious-input case per ingestion path (bad/missing CRS
  metadata, oversized file, malformed XML), not only correctness cases on
  well-formed input — the security risk on this boundary is adversarial or
  malformed input, not just an honest CRS mismatch.

## Positions

- AGREE: NEVER hardcode credentials, API keys, or secrets (already drafted
  in `discovered-rules.md`'s Forbidden section) — correct as stated; I've
  proposed the concrete enforcement mechanism (gitleaks/detect-secrets +
  Secret Manager) above rather than objecting to the rule itself.
- AGREE: the "public data only, no identity/permission model needed"
  resolution of the C-05 contradiction (`evidence.md`'s Contradiction Noted
  section) is correctly derived from the scope document for the **read/view**
  path.
- OBJECT: the draft's "no identity or permission model is required for the
  demo" is stated as covering the whole system, but no artifact I read says
  whether the **upload/ingestion endpoint** is also unauthenticated. An
  unauthenticated public write path that accepts arbitrary files is a
  materially different risk from an unauthenticated read path over public
  data — it exposes the USD 100/month budget to storage/compute exhaustion
  and the system to malicious file uploads regardless of whether the data
  itself is public. This should be a named open point for the interview, not
  assumed settled by the C-05 resolution.
- AGREE: the code-style tool suggestions (Black/Ruff, Prettier/ESLint) are
  reasonable defaults; I've added that they must be CI-enforced, not just
  locally run, which the draft leaves implicit.
- AGREE: `poc` scope adding no extra coverage floor per `org.md` is correctly
  flagged as an open question pending scope-classification confirmation; my
  security-test addition above is scoped as an addition to the targeted
  regression floor the lead already proposed, not a new coverage-floor
  demand.

### Open points for the human interview (devsecops-specific, not yet covered by the lead draft)

- Is the ingestion/upload endpoint authenticated, or is it also open to
  unauthenticated public use like the read/view path? (See OBJECT above.)
- What repository host will this project use (GitHub, GitLab, other)? This
  determines whether free-tier native scanning (e.g., GitHub Dependabot,
  GitHub secret scanning) is available alongside or instead of gitleaks/
  Trivy/pip-audit.
- Should database and third-party API credentials (Cloud SQL, the
  commercial geocoding API key from D-03) be sourced from GCP Secret
  Manager from day one, or is a plain `.env`-only approach acceptable until
  the GCP account is confirmed (C-10)?
- Does the commercial geocoding API (D-03) receive any data that could
  indirectly identify a person or a research subject's location (e.g.,
  addresses tied to survey responses), even though C-11 records no personal
  data is expected? Worth a one-line confirmation given the regulatory
  register already treats this as a live open question if it ever changes.
- Is a pre-commit hook step (for lint/format/secret-scanning) acceptable
  ceremony for a 1-2 developer team, or should these checks run in CI only?
