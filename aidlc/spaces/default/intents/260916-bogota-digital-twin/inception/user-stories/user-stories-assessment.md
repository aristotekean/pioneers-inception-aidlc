# User Stories Assessment

Upstream: `requirements.md` (Requirements Analysis), `team-practices.md` (Practices Discovery).

## Decision

**Execute.** User stories add value for this proof of concept.

## Rationale

- The work is user-facing end to end: four researchers use a map interface to upload data, explore layers, inspect features and switch to a 3D view (requirements FR2, FR4, FR5, FR7).
- Test-first development is affirmed for every layer (`team-practices.md`), so acceptance criteria written now become the tests written first in construction.
- The requirements carry decisions that changed late (per-user login, per-layer visibility); stories make the resulting user journeys explicit and independently testable before design and units are cut.

## Factors Considered

| Factor | Signal |
|---|---|
| Project type | Greenfield proof of concept, web application |
| User-facing scope | Every must-have is a screen or interaction (upload wizard, layer panel, popup, 2D/3D toggle, sign-in) |
| Personas | One role, but users come from a university, a private company and a public-sector agency with different data |
| Complexity signals | Geometry detection paths, per-layer visibility, background jobs with per-row failure reporting, 3D pilot boundary |
| Cross-team work | Single small team; stories still needed for testability, not coordination |

## Where Stories Add the Most Value

- The upload wizard's detection and failure paths (FR2.1 to FR2.8), where the acceptance criteria define the error behaviour the affirmed hard rule demands.
- Layer visibility and deletion (FR4.5, FR4.6), where the late login decision needs concrete journeys.
- The 3D pilot boundary (FR5.2, FR5.3, FR5.5), where "inside" and "outside" the pilot area must be testable.
