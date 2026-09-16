# User Flows: Bogotá Urban Digital Twin

Upstream: `intent-statement.md`, `scope-document.md`, `intent-backlog.md`. Answers: `rough-mockups-questions.md`. Persona for every flow: one of the four researchers (intent-statement). There is no login for the proof of concept (scope-document).

## Flow 1: Load a dataset and see it on the map (core happy path)

```
Flow: Load a dataset as a layer
Persona: Researcher
Trigger: The researcher has a CSV or JSON file with coordinates
Steps:
  1. Main map view -> clicks "Agregar datos" -> Add data wizard opens at step 1
  2. Step 1 -> names the layer, chooses the file, clicks "Siguiente" -> file is read, columns and geometry detected
  3. Step 2 -> reviews detected geometry and visible columns, adjusts if needed, clicks "Siguiente" -> summary shown
  4. Step 3 -> clicks "Cargar" -> wizard closes, layer appears in the list as loading, status bar shows 1 job
  5. Main map view -> waits -> layer becomes active, map zooms to its extent
Success outcome: The new layer is listed, visible and toggleable; clicking a feature shows its attributes
Error paths:
  - Unreadable file -> step 1 error message -> choose another file
  - No geometry detected -> step 2 blocks "Siguiente" with a hint -> choose the geometry column manually or cancel
  - Load job fails -> layer shows a warning in the list -> "Ver detalle" explains the failure; the researcher fixes the file and retries
```

Sources: [Q1] [Q2]; backlog PU-01, PU-04.

```
[Mapa] --Agregar datos--> [Paso 1: archivo] --Siguiente--> [Paso 2: columnas]
                                                              |
                                              sin geometría --+--> (hint, bloqueado)
                                                              |
                                                          Siguiente
                                                              v
                                                    [Paso 3: confirmar] --Cargar--> [Mapa: capa cargando]
                                                                                        |
                                                                            éxito ------+------ error
                                                                              v                   v
                                                                 [Mapa: capa activa]   [Lista: advertencia]
```

## Flow 2: Explore the official cadastre

```
Flow: Explore cadastral parcels
Persona: Researcher
Trigger: The researcher wants to check a parcel's attributes
Steps:
  1. Main map view -> enables "Predios catastrales" in the layer list -> parcels render on the map
  2. Map -> zooms and pans to the area of interest -> parcels remain visible at the working scale
  3. Map -> clicks a parcel -> popup shows its attributes (code, area, floors, height, localidad)
  4. Popup -> presses Escape or clicks elsewhere -> popup closes
Success outcome: The researcher reads the parcel's attributes without leaving the map
Error paths:
  - Attributes fail to load -> popup shows a retry link -> retry
  - Parcel layer fails to load -> warning in the layer list -> the researcher reports it at the next milestone demo
```

Sources: [Q3]; backlog PU-03, PU-05; intent-statement success metric (cadastre visible and inspectable).

## Flow 3: Switch to 3D over the pilot localidad

```
Flow: View terrain and buildings in 3D
Persona: Researcher
Trigger: The researcher wants to see heights and relief in the pilot area
Steps:
  1. Main map view -> clicks "3D" in the toggle -> canvas switches to the 3D view, terrain tiles load progressively
  2. 3D view -> pans to the pilot localidad -> buildings appear extruded with their heights
  3. 3D view -> tilts and orbits -> relief and heights are visible against the cadastre and uploaded layers
  4. 3D view -> clicks a building -> popup shows the parcel attributes including height
  5. 3D view -> clicks "2D" -> returns to the flat map at the same location
Success outcome: Terrain and building heights are visible for the pilot localidad; the researcher returns to 2D without losing position
Error paths:
  - Terrain unavailable -> falls back to 2D with a message -> retry later
  - Outside the pilot localidad -> flat terrain and no buildings, banner explains the pilot boundary
```

Sources: [Q4]; backlog PU-02; intent-statement success metric (3D terrain and heights).

## Flow 4: Toggle and style layers

```
Flow: Adjust the map to compare layers
Persona: Researcher
Trigger: Too many layers or the wrong colours make the map hard to read
Steps:
  1. Main map view -> unchecks a layer -> it disappears from the map immediately
  2. Layer list -> opens "estilo" on a layer -> colour, opacity and point size controls appear inline
  3. Style controls -> changes colour -> the map updates live
  4. Layer list -> collapses the side panel -> the map takes the full width
Success outcome: The researcher sees only the layers they want, in distinguishable colours
Error paths:
  - None specific; style changes are local and reversible
```

Sources: [Q1]; backlog PU-05; scope-document must-have (layer panel).

## Flow Map

```
                      +-----------------------+
                      |    Main map view (2D) |
                      +-----+-----+-----+-----+
                            |     |     |
        Agregar datos <-----+     |     +-----> 2D/3D toggle
              |                   |                    |
              v                   v                    v
     [Add data wizard]     [Feature popup]        [3D view]
              |                                        |
              +------> new layer in list <-------------+ (same panel, same popup)
```

## Assumptions & Open Questions

- The map remembers the last position and layer visibility within a session only; persistence across sessions is not assumed for the proof of concept. [assumption]
