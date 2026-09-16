# Intent Statement: Bogotá Urban Digital Twin

## Problem Statement

Researchers working on Bogotá have no single web-based place to bring external data sources (CSV, JSON, other databases, external APIs) together as layers on a map of the city, explore them, and later run analyses such as flood-risk simulation on top of cadastral and terrain data. [desc]

The initiative is a proof of concept: its purpose is to validate the technical approach for an urban digital twin of Bogotá before committing further investment. [Q1]

The first domain is cadastre (catastro), and the first analytical use case is flood-risk simulation built on cadastral and terrain data. [desc]

## Target Customer

The initial users are four internal researchers; the tool is not public-facing and access control is not required initially. [desc]

The four researchers form a mixed group drawn from a university, a private company and a public-sector agency. [Q2]

They benefit by being able to load their own datasets, see them on a shared map of Bogotá, inspect attributes, and eventually visualize simulation results as new layers. [desc]

## Success Metrics

The first version is successful when all of the following are demonstrable: [Q3]

| Metric | Pass criterion | Source |
|---|---|---|
| Self-serve dataset load | A new CSV/JSON dataset can be loaded and shown as a map layer without developer intervention | [Q3] |
| Full cadastre on the map | Official IDECA cadastral parcels for all of Bogotá are visible and inspectable on the map | [Q3] |
| 3D terrain and building heights | Terrain and LoD1 building heights are rendered in 3D from the LiDAR-derived products | [Q3] |

A first flood-risk scenario is not a success metric for this piece of work: the flood model is exploratory, and success is judged on roadmap phases 1 to 4 only. [Q9]

The first demonstrable milestone covering roadmap phases 1 to 4 is targeted at about 3 to 6 months. [Q10]

Progress is shown through milestone demos at the end of each roadmap phase. [Q7]

## Initiative Trigger

Three triggers coincide: a funded research project or grant with defined deliverables, the availability of open IDECA LiDAR-derived products (terrain and surface models, orthophotos, cadastre) that make the work feasible now, and an institutional opportunity or partnership with an agency, university, or funding body. [Q4]

## Initial Scope Signal

- Workflow-selected scope: `mvp` (workflow-selected). [scope]
- User-confirmed product boundary: the full roadmap through phase 7, that is, up to and including a first flood-risk model whose results appear as layers on the map. [Q8]
- Within that boundary the flood model is exploratory; the measurable commitment is roadmap phases 1 to 4 (cadastral layer on the map, self-serve upload and layer panel, terrain, building heights and the LoD1 3D city view). [Q9]
- Scope and priority decisions are made by the four researchers by consensus. [Q6]
- Explicitly out of scope per the initial description: virtual reality, edge/IoT computing with sensors, and generative geometry; access control is deferred until the tool opens to more users. [desc]

## Assumptions & Open Questions

None.
