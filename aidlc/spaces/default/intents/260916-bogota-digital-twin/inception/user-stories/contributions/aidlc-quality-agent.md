**Collaborator:** aidlc-quality-agent

## Contribution

### 1. NFR1, NFR2, NFR3 are not reflected as testable anywhere in `stories.md`

- **NFR1** (measure-and-report tile latency, city scale, 4 concurrent users) has no story or acceptance criterion at all. Per the affirmed project correction ("record a 'best effort' answer as a measure-and-report requirement... so the requirement still has a pass/fail check"), this needs either a concrete AC now or an explicit `Deferred` traceability row naming the stage that will own it (e.g. `nfr-design` or `performance-validation`). Recommend adding to Group 6 (Operational visibility), e.g.:
  - `US6.3 Report cadastre tile latency` — As Andrés, I want the median and 95th-percentile tile response time for the cadastre layer recorded under 4 concurrent users, so that a real threshold can be set later.
    - `AC6.3.1` Given the cadastre layer under a 4-concurrent-user load test, when the run finishes, then the median and p95 tile response times are written to a report artifact.
- **NFR2** (10 MB CSV load completes within 5 minutes on the shared environment) is untested — `US2.6`'s acceptance criteria (`AC2.6.1`–`AC2.6.3`) cover state transitions but never assert a time bound. Recommend adding, e.g.: `AC2.6.4 Given a 10 MB CSV on the shared environment, when the load runs, then it completes and the layer becomes active within 5 minutes.` This is the one NFR in this set with a ready-made fixture (a 10 MB CSV) and a numeric threshold — no reason to leave it untested.
- **NFR3** (password hashing, HTTPS) — `US1.3` traces NFR3, but `AC1.3.1`/`AC1.3.2` only test rejection of unauthenticated requests; neither hashing nor HTTPS enforcement is asserted anywhere. Recommend either adding two criteria under `US1.1` (e.g. "Given a seeded account, when its stored credential is inspected, then it is a hash, never the plaintext password" and "Given any request over plain HTTP, when it reaches the service, then it is redirected or rejected") or explicitly marking these two sub-requirements `Deferred` to `nfr-design`/`infrastructure-design` in `traceability.json` with that named target — leaving them silently uncovered is the gap, not the choice of stage.

### 2. `AC1.2.1` lacks a measurable value (session lifetime)

`AC1.2.1`: "Given I signed in, when I reload the page within the session lifetime, then I remain signed in." — "session lifetime" is called out as an open `[assumption]` in both `personas.md`'s and `stories.md`'s Assumptions sections, deferred to design. As written, no fixture can exist for this criterion yet: a test cannot assert "within the session lifetime" without a number. Recommend either a working default now (e.g. "within a 24-hour session") so a fixture can be built today, or an explicit `Deferred` traceability row to whichever design stage sets the value — not a silent open assumption sitting under a criterion the team is committing to test-first.

### 3. Missing round-trip reprojection test (team-affirmed practice)

`team.md` §Testing Posture is explicit: "plus one round-trip test (reproject forward, then inverse-reproject) to catch transform-direction and datum-mismatch bugs." `stories.md` has `AC2.9.1` (forward EPSG:9377 → EPSG:4326 reference point) and `AC2.9.2` (identity check for already-WGS84 CSV) but no round-trip criterion. Recommend adding `AC2.9.4`: "Given a loaded geometry reprojected from EPSG:9377, when it is reprojected back to EPSG:9377, then the result is within tolerance of the original input coordinate." Without it, a transform-direction bug (e.g. an axis swap that happens to land close to the expected point) could pass `AC2.9.1` alone.

### 4. GeoJSON path has no "no geometry detected" negative case symmetric to the CSV path

FR2.2 states the detection order applies to native geometry *and* coordinate columns, and "when none is found the wizard blocks progress." The CSV path has this negative case (`AC2.4.1`: no recognisable coordinate columns → blocked, hint shown). The GeoJSON path (`US2.2`) only has `AC2.2.2`, which covers a feature *within* a valid FeatureCollection lacking geometry (a warning, load still proceeds) — not a file where geometry detection fails entirely (e.g. a JSON file that is not a valid FeatureCollection, or one with zero features carrying any geometry). Recommend adding under `US2.2` or `US2.4`, e.g.: `AC2.2.3 Given a JSON file with no Feature geometries and no recognisable coordinate columns, when step 2 opens, then no geometry source is selected, "Siguiente" is disabled, and a hint asks me to choose columns or a different file.`

### 5. Stories with happy-path-only acceptance criteria (Must Have priority)

The following Must Have stories have no error or edge-case criterion at all; flagging so the lead can judge whether each needs one or is genuinely low-risk enough to skip:
- `US1.2` (session persist/sign-out) — no case for an already-expired session being reloaded.
- `US4.1` (toggle layers) — no case for a layer that fails to render/toggle.
- `US4.2` (style a layer) — no case for an invalid style value.
- `US4.4` (status bar) — no case for zero running jobs or a stalled job count.
- `US5.1` (2D/3D toggle) — no case for toggling while a layer fetch is in flight.
- `US5.4` (layers visible in 3D) — no case for a layer that fails to project onto terrain.
- `US2.10` (zoom to new layer) — no case for an empty-extent layer (e.g. all rows failed but job still reports partial success).
- `US2.8` (keep original file) — no case for a file-storage write failure during the keep step.

None of these block the gate on their own, but per the phase-inception rule ("each requirement must have a clear pass/fail criterion") and the team's TDD-everywhere posture, at least the Must Have ones in map-state-changing paths (`US4.1`, `US5.1`) are worth one edge case each before Code Generation writes their first failing test.

### 6. FR/NFR → story coverage check

Cross-checked every `FR` and `NFR` ID in `requirements.md` against a `Traces:` line in `stories.md`:

- **FR1–FR5, FR7**: every sub-ID (FR1.1–FR1.4, FR2.1–FR2.8, FR3.1–FR3.3, FR4.1–FR4.7, FR5.1–FR5.5, FR7.1–FR7.2) has at least one story naming it. No gaps.
- **FR6**: FR6.1 → `US6.1`, FR6.2 → `US6.2`. **FR6.3 ("show per-job status and failure detail in the interface") is never named in any story's `Traces:` line**, even though it is functionally delivered by `US2.6`/`US2.7`. This is a traceability gap, not a functional one — recommend the lead add `FR6.3` to the `Traces:` line of `US2.6` and/or `US2.7`, or give `traceability.json` an explicit `OK` row for `FR6.3` targeting those two stories, so the `traceability` sensor doesn't have to infer the link.
- **NFR1, NFR2, NFR3**: see item 1 above — currently no story traces any of them, so `traceability.json` will need `Deferred` or `GAP` rows unless the ACs above are added first.
- **NFR4, NFR5, NFR6, NFR8, NFR9**: not user-facing functional behavior (data-protection policy, cost ceiling, the team's own testing practice, accessibility with no formal target, availability with no redundancy requirement) — reasonable to mark `N/A` in `traceability.json` with a one-line justification each, not a story gap.
- **NFR7** (observability = FR6.1 + FR6.2 + FR6.3): covered by `US6.1`/`US6.2` plus the FR6.3 fix above.

**Uncovered IDs to resolve in `traceability.json` (as `Deferred`/`GAP`/`N/A`, or covered by the ACs proposed above): NFR1, NFR2, NFR3, and the FR6.3 traceability-line gap.**

## Positions

- OBJECT: NFR1, NFR2 and NFR3 have no testable acceptance criterion anywhere in `stories.md` — the project's own affirmed correction requires a measure-and-report pass/fail check for exactly this kind of "best effort" answer, and NFR2 in particular has a ready numeric threshold (10 MB / 5 minutes) that should not go untested.
- OBJECT: `AC1.2.1`'s "session lifetime" has no measurable value, so no fixture can be written for it yet under test-first; it should get a working default or an explicit `Deferred` traceability target.
- OBJECT: the reprojection acceptance criteria (`AC2.9.1`–`AC2.9.3`) omit the round-trip test that `team.md` explicitly mandates for catching datum-mismatch and transform-direction bugs.
- OBJECT: the GeoJSON geometry-detection path lacks a "nothing detected, block progress" negative case that the CSV path already has (`AC2.4.1`), even though FR2.2 states the same detection-order requirement applies to both.
- AGREE: the required negative cases called out for this review — no geometry detected (CSV), invalid coordinates, unsupported coordinate system, unauthenticated access, and private-layer tile access — are all present with clear Given/When/Then pass/fail wording (`AC2.4.1`, `AC2.3.2`/`AC2.7.1`, `AC2.9.3`, `AC1.3.1`/`AC1.3.2`, `AC4.6.2`).
- AGREE: the reference-point-with-tolerance pattern (`AC2.9.1`, `AC3.1.2`, `AC5.2.1`, `AC5.2.2`) is applied consistently and matches the team's affirmed spatial-test fixture approach well.
- AGREE: FR1 through FR5 and FR7 are fully traced to stories with no gaps at the sub-requirement level.
