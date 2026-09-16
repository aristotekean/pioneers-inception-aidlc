# User Stories Plan and Questions

## Story Plan

- **Persona approach**: one role (administrator rights for all four researchers), so personas are distinguished by what they bring and want, not by permissions. Candidates: a university researcher who brings survey files, an agency researcher who brings official datasets, a company researcher who consumes results in 3D. The exact set is decided in Q1.
- **Story format**: "As a [persona], I want [goal], so that [benefit]" with INVEST checks; every story gets a stable `US{group}.{seq}` ID and every acceptance criterion an `AC{group}.{seq}.{n}` ID in Given/When/Then form, covering the happy path and at least one error or edge case, so they become the first tests written under the affirmed test-first practice.
- **Prioritization**: MoSCoW from the requirements (must-haves = FR1 to FR7 as scoped; should-haves = geocoding, identity extras; could-haves = connector, flood model). The MVP boundary is formally decided at Delivery Planning.
- **Breakdown approach**: decided in Q2.

## Q1. How should the personas be defined?

A. One persona, "the researcher": one role, one set of goals
B. Three personas by institution (university, public-sector agency, private company), differing in the data they bring and what they want from the map
C. Three personas by activity (data owner who uploads, explorer who inspects layers, analyst who works in 3D)
D. Not yet defined
X. Other (please specify)

[Answer]: B. Three personas by institution (university, public-sector agency, private company), differing in the data they bring and what they want from the map

## Q2. How should the stories be grouped?

A. By feature area, mirroring the requirement groups (accounts, upload, cadastre, layer panel, 3D, operations)
B. By user workflow, mirroring the four flows in the wireframes (load a dataset, explore the cadastre, switch to 3D, adjust layers)
C. By backlog unit (PU-01 end-to-end slice, PU-02 3D, PU-03 cadastre, PU-04 upload, PU-05 layer panel)
D. Not yet defined
X. Other (please specify)

[Answer]: A. By feature area, mirroring the requirement groups (accounts, upload, cadastre, layer panel, 3D, operations)

## Q3. How fine should the stories be?

A. Fine: each story is 1 to 3 days of work, roughly 20 to 30 stories
B. Coarse: each story is a feature, roughly 8 to 12 stories, with more acceptance criteria each
C. Not yet defined
X. Other (please specify)

[Answer]: A. Fine: each story is 1 to 3 days of work, roughly 20 to 30 stories

## Q4. Should later capabilities (geocoding, external API connector, flood model) get stories now?

A. Yes, at title level only, marked Should Have or Could Have, without acceptance criteria
B. No; only the must-haves for this piece of work get stories
C. Not yet defined
X. Other (please specify)

[Answer]: A. Yes, at title level only, marked Should Have or Could Have, without acceptance criteria

## Consolidated Summary Confirmation

- Personas: three, by institution (university, public-sector agency, private company), same role, different data and goals
- Grouping: by feature area, mirroring the requirement groups
- Granularity: fine, 1 to 3 days each, roughly 20 to 30 stories
- Later capabilities: title-level stories marked Should Have or Could Have, no acceptance criteria

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
