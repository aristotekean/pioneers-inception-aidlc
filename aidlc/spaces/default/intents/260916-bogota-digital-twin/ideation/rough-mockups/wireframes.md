# Wireframes: Bogotá Urban Digital Twin (low fidelity)

Upstream: `intent-statement.md`, `scope-document.md`, `intent-backlog.md`. Answers: `rough-mockups-questions.md`.

Low-fidelity concept wireframes for the proof of concept. The interface language is Spanish [Q8]; labels below are the Spanish labels the researcher would see, with an English gloss in parentheses where helpful. Desktop and tablet are the target form factors [Q6]; no brand guidelines apply, so the styling is neutral [Q5]. No formal accessibility conformance is required for the proof of concept [Q7]; each screen still carries a one-line accessibility note as the stage requires.

## Information Architecture

The application is map-first: a single full-screen map with a collapsible side panel [Q1]. There is no login and no dataset page for the proof of concept (scope-document).

```
Mapa (single screen)
 ├── Barra superior: título, alternador 2D/3D, ayuda
 ├── Panel lateral (colapsable)
 │    ├── Capas (layer list, toggles, styling)
 │    └── Agregar datos (wizard, opens over the panel)
 ├── Lienzo del mapa (map canvas)
 │    └── Ventana emergente de atributos (feature popup)
 └── Barra de estado: coordenadas del cursor, escala, trabajos en curso
```

## Screen 1: Main Map View (2D, populated)

The default state after loading: 2D map of Bogotá with the official cadastre layer and any uploaded layers listed in the side panel [Q1] [Q4].

```
+------------------------------------------------------------------------------+
| Gemelo Digital Bogotá                          [ 2D | 3D ]      [ ? Ayuda ]  |
+------------------+-----------------------------------------------------------+
| CAPAS        [<] |                                                           |
|                  |                                                           |
| [x] Predios      |               (mapa 2D de Bogotá)                        |
|     catastrales  |                                                           |
|     ● ─ estilo   |                    ┌──────────┐                           |
| [x] Mis puntos   |                    │ predio   │                           |
|     de muestreo  |                    │ ▪ ▪ ▪    │                           |
|     ● ─ estilo   |                    └──────────┘                           |
| [ ] Manzanas     |                                                           |
|     ● ─ estilo   |                                                           |
|                  |                                                           |
| [+ Agregar datos]|                                                           |
|                  |                                                           |
+------------------+-----------------------------------------------------------+
| 4.6097, -74.0817 | escala 1:10 000 | Trabajos: ninguno en curso              |
+------------------------------------------------------------------------------+
```

Elements:
- Layer list with a checkbox to toggle each layer, a colour swatch and an "estilo" (style) control for basic styling [Q2 scope must-have: layer panel].
- "Agregar datos" (add data) button, the single primary action of the panel [Q2].
- 2D/3D toggle in the top bar, 2D selected by default [Q4].
- Status bar with cursor coordinates, scale and background job status.

Accessibility note: h1 = "Gemelo Digital Bogotá" in the header landmark; side panel is a complementary landmark, map canvas is main; keyboard entry point is the layer list (Tab from the header), arrow keys move between layers, Space toggles.

## Screen 1 states

- **Empty** (first run, no layers): the layer list shows "Aún no hay capas. Agrega tus datos o carga el catastro oficial." with the "Agregar datos" button as the only action; the map shows the base map of Bogotá centred on the city.
- **Loading**: a layer being loaded shows a spinner beside its name and a greyed checkbox; the status bar reads "Trabajos: 1 en curso".
- **Error**: a layer that failed to load shows a warning icon and the text "No se pudo cargar. Ver detalle" that opens the job detail; the map is unaffected.
- **Partial**: many layers (20+) make the list scroll inside the panel; long layer names wrap to two lines; a layer with zero features shows "0 elementos" beside its name.

## Screen 2: Feature Popup

Clicking a feature opens a popup anchored at the click location showing the feature's attributes [Q3].

```
                     ┌────────────────────────────────────┐
                     │ Predio catastral              [x]  │
                     ├────────────────────────────────────┤
                     │ Código        0012345678           │
                     │ Área (m²)     240                  │
                     │ Pisos         3                    │
                     │ Altura (m)    9.4                  │
                     │ Localidad     Teusaquillo          │
                     │ ...           (desplazable)        │
                     ├────────────────────────────────────┤
                     │ Capa: Predios catastrales          │
                     └────────────────────────────────────┘
                              ▼ (anchored at the clicked feature)
```

Elements:
- Title = layer's display name; attribute table = the layer's visible fields in the order defined at upload.
- Long attribute lists scroll inside the popup; the popup never covers the side panel.
- Clicking elsewhere on the map or pressing Escape closes it.

Accessibility note: popup is a dialog with h2 = feature title; focus moves into the popup on open and returns to the map on close; Escape closes.

States: **loading** shows a skeleton of three rows while attributes are fetched; **error** shows "No se pudieron obtener los atributos" with a "Reintentar" (retry) link; **partial** shows "Sin valor" for empty attributes rather than a blank cell.

## Screen 3: Add Data Wizard

Opened from "Agregar datos"; three steps over the side panel [Q2]. Files are CSV or JSON for the proof of concept (scope-document).

```
Step 1 of 3 — Archivo
+------------------+
| Agregar datos    |
| ● ○ ○  Paso 1/3  |
|                  |
| Nombre de capa   |
| [______________] |
|                  |
| Archivo          |
| [ Elegir CSV o   |
|   JSON ...     ] |
|  o arrastrar aquí|
|                  |
|        [Siguiente]|
+------------------+

Step 2 of 3 — Columnas y geometría
+------------------+
| ○ ● ○  Paso 2/3  |
|                  |
| Detectamos 8     |
| columnas.        |
|                  |
| Geometría        |
| (•) lat/lon:     |
|     latitud,     |
|     longitud     |
| ( ) columna WKT  |
| ( ) GeoJSON      |
|                  |
| Columnas visibles|
| [x] nombre       |
| [x] tipo         |
| [ ] id_interno   |
|                  |
| [Atrás] [Siguiente]|
+------------------+

Step 3 of 3 — Confirmar
+------------------+
| ○ ○ ●  Paso 3/3  |
|                  |
| Capa: Mis puntos |
| Archivo: p.csv   |
| Filas: 1 240     |
| Geometría:       |
|   lat/lon        |
| Sistema de coord.|
|   WGS84 (asumido)|
|                  |
| [Atrás] [Cargar] |
+------------------+
```

After "Cargar" (load), the wizard closes, the new layer appears in the list in the loading state, and the map zooms to its extent when the job completes.

Elements:
- Step 2 shows what was detected and lets the researcher override the geometry choice; the detection order (native geometry, then coordinate columns) follows the initial description. Reference codes and addresses are a later capability (scope-document) and are not offered here.
- Step 3 is a read-only summary; "Cargar" is the only primary action.

Accessibility note: the wizard is a dialog with h2 = "Agregar datos"; focus is trapped inside; each step's first field receives focus; Escape asks to discard.

States: **error** at step 1 for an unreadable file ("No pudimos leer el archivo. Verifica que sea CSV o JSON."); **partial** at step 2 when no geometry is detected (all geometry radios unselected, "Siguiente" disabled, hint "Elige la columna con la geometría"); **loading** after "Cargar" is shown in the layer list, not in the wizard.

## Screen 4: 3D View (pilot localidad)

The 2D/3D toggle switches the canvas to terrain and extruded buildings for the pilot localidad [Q4]; the side panel and popup behave the same.

```
+------------------------------------------------------------------------------+
| Gemelo Digital Bogotá                          [ 2D | 3D ]      [ ? Ayuda ]  |
+------------------+-----------------------------------------------------------+
| CAPAS        [<] |          (vista 3D inclinada)                             |
|                  |                                                           |
| [x] Terreno      |                  ▲▲▲                                      |
| [x] Edificios 3D |            ▄▄  ▲▲▲▲▲  ▄▄▄                                 |
| [x] Predios      |         ▄▄ ██ ▄▄▄▄▄▄▄ ███ ▄▄                              |
|     catastrales  |      ▄▄▄ ██ ███ ▄▄▄ ███ ███ ▄▄                            |
| [x] Mis puntos   |   ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄                          |
|                  |                                                           |
| [+ Agregar datos]|   ⓘ Vista 3D disponible para la localidad piloto          |
+------------------+-----------------------------------------------------------+
| 4.6097, -74.0817 | inclinación 45° | Trabajos: ninguno en curso              |
+------------------------------------------------------------------------------+
```

Elements:
- "Terreno" (terrain) and "Edificios 3D" (3D buildings) appear as layers only in 3D mode.
- An information banner states the 3D view covers the pilot localidad (scope-document, pilot area).
- Navigation: drag to pan, scroll to zoom, right-drag or keyboard to tilt.

Accessibility note: same landmarks as Screen 1; the toggle is a two-option radio group announced as "Vista 2D" / "Vista 3D"; keyboard camera control via arrow keys with Shift for tilt.

States: **loading** shows the terrain tiles resolving progressively with the banner "Cargando terreno..."; **error** falls back to the 2D view with the message "La vista 3D no está disponible ahora"; **partial** outside the pilot localidad shows flat terrain and no buildings, with the banner explaining the pilot boundary.

## Screen Inventory

| Screen | Purpose | Backlog | Source |
|---|---|---|---|
| Main map view (2D) | Explore layers, toggle and style them | PU-01, PU-03, PU-05 | [Q1] [Q4] |
| Feature popup | Inspect attributes of a clicked feature | PU-05 | [Q3] |
| Add data wizard | Self-serve upload with detection and confirmation | PU-01, PU-04 | [Q2] |
| 3D view | Terrain and building heights for the pilot localidad | PU-02 | [Q4] |

## Assumptions & Open Questions

- The base map underneath the layers is a neutral street map; the orthophoto basemap is out of scope (scope-document). [assumption]
- Layer styling in the proof of concept is limited to colour, opacity and point size; richer symbology is not shown in these wireframes. [assumption]
