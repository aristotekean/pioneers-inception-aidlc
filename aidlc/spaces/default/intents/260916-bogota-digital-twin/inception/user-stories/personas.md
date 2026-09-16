# Personas: Bogotá Urban Digital Twin

Upstream: `requirements.md`, `team-practices.md`; plan answers in `user-stories-questions.md` (three personas by institution, Q1). All three share the single administrator role (FR1.2); they differ in the data they bring and what they want from the map.

## Camila, university researcher

| Field | Value |
|---|---|
| Role | Researcher at a university group; administrator rights like everyone |
| Goals | Load her own field survey files (CSV with latitude/longitude) and see them against the official cadastre; inspect attributes of the parcels her points fall on; share a layer with the other three |
| Pain points | Today her data lives in spreadsheets and desktop GIS projects nobody else can open; every comparison against official data is a manual export |
| Tech comfort | Medium: comfortable with spreadsheets and a desktop GIS, not with servers |
| Frequency | Several times a week during a field campaign |
| Devices | Laptop; sometimes a tablet in the field |

## Andrés, public-sector agency researcher

| Field | Value |
|---|---|
| Role | Researcher at a district agency; the team's link to official data and to institutional restrictions |
| Goals | Get the official cadastre and terrain products onto the shared map correctly (right coordinate system, right attributes); control who sees an agency dataset he uploads; trust that a load either succeeded row by row or told him what failed |
| Pain points | Coordinate mismatches that silently misplace data; datasets that cannot be shared outside the agency without a visibility control; no audit of what was loaded when |
| Tech comfort | High in GIS, medium in web tools |
| Frequency | Weekly; heavier at the start when official data is loaded |
| Devices | Desktop workstation |

## Laura, private-company researcher

| Field | Value |
|---|---|
| Role | Researcher at a company partner; consumes the assembled map more than she uploads |
| Goals | Explore the pilot localidad in 3D with real terrain and building heights; toggle and restyle layers to compare them; read attributes quickly without leaving the map |
| Pain points | Flat maps hide the relief that matters for flood questions; too many layers in one colour; slow or broken 3D views |
| Tech comfort | Medium |
| Frequency | Weekly reviews and milestone demos |
| Devices | Laptop and tablet during demos |

## Persona Relationships and Priority

| Priority | Persona | Why |
|---|---|---|
| 1 | Andrés | Owns the official data loads that every success metric depends on (cadastre, terrain) and the visibility decisions |
| 2 | Camila | Exercises the self-serve upload path end to end, the first success metric |
| 3 | Laura | Exercises exploration and the 3D success metric |

All three see each other's public layers; a private layer is visible only to the users its uploader chose (FR4.6).

## Assumptions & Open Questions

- The names are placeholders for the three institutional profiles the user confirmed; the real researchers are not named in any artifact. [assumption]
