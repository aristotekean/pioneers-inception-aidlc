# Interaction Specification: Bogotá Urban Digital Twin

Upstream: `mockups.md`, `wireframes.md`, `user-flow.md`, `stories.md`, `requirements.md`, `team-practices.md`. Format follows the component specification template. Labels are the Spanish UI strings; prose is English.

## Layer Row

| Field | Value |
|---|---|
| Component | LayerRow |
| Description | One layer in the side panel: toggle, swatch, name, status, actions menu, optional inline style control |
| Category | display / input |

### States

| State | Description | Trigger |
|---|---|---|
| default | Checkbox reflects visibility, swatch shows colour | list render |
| hover / focus | Row highlighted, menu button visible | pointer or Tab |
| loading | Spinner replaces swatch, checkbox disabled, text "cargando" | job running |
| error | Warning icon, text "N filas con error · Ver detalle" or "No se pudo cargar · Ver detalle" | job finished with failures |
| empty | Text "0 elementos" beside the name | layer has zero features |
| style-open | Inline style panel expanded under the row | "Estilo" chosen |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| layer | object | yes | — | id, name, geometryType, style, visible, jobStatus, failedRows, featureCount, ownerId |
| isOwner | boolean | yes | — | Shows "Visibilidad" and "Eliminar" only when true |
| onToggle | function | yes | — | Emits visible boolean |
| onStyleChange | function | yes | — | Emits {color, opacity, pointSize} live |
| onOpenMenu | function | yes | — | Opens the actions menu |

### Responsive Behaviour

| Breakpoint | Behaviour |
|---|---|
| tablet (768 to 1023 px) | Row height 48 px minimum; menu button always visible (no hover) |
| desktop (1024 px and above) | Row height 40 px; menu button on hover or focus |

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | listitem inside a list; checkbox is a native input with the layer name as label |
| Keyboard interaction | Tab to row, Space toggles, Enter opens menu, Escape closes menu or style panel |
| Label | Visible name; status text read after the name |
| Contrast ratio | 4.5:1 for text; swatch has a 1 px border so light colours remain visible |
| Screen reader | "Predios catastrales, capa, visible" then status |
| Focus management | Focus returns to the menu button when the menu closes |

## Actions Menu

| Field | Value |
|---|---|
| Component | LayerActionsMenu |
| Description | Three-dot menu with "Estilo", "Visibilidad", "Eliminar" |
| Category | navigation |

States: closed, open; "Visibilidad" and "Eliminar" absent for non-owners (AC4.6.3). Keyboard: arrow keys move, Enter selects, Escape closes, focus returns to the trigger. Role: menu with menuitems.

## Style Control (assumption: inline)

| Field | Value |
|---|---|
| Component | LayerStylePanel |
| Description | Colour, opacity and point size controls applied live |
| Category | input |

Inputs: color (native color input), opacity (range 0 to 100, step 5, live label "Opacidad 70 %"), pointSize (range 1 to 12, only for point layers). Changes emit immediately (AC4.2.1) and persist per user (AC4.2.3). Keyboard: arrows on the ranges; each input has a visible label.

## Add-data Wizard

| Field | Value |
|---|---|
| Component | AddDataWizard |
| Description | Three-step upload flow inside the side panel |
| Category | input / feedback |

### States

| State | Description | Trigger |
|---|---|---|
| step1-default | Name empty, no file; "Siguiente" disabled | open |
| step1-ready | Name and valid file present | valid input |
| step1-error | Inline error: size or format | invalid file |
| step2-detected | Geometry pre-selected, counts shown, optional warnings | detection result |
| step2-blocked | No geometry; "Siguiente" disabled; manual column selects | nothing detected |
| step3-summary | Read-only summary plus visibility choice | step 2 confirmed |
| cancel-confirm | "¿Descartar la carga?" dialog | Cancelar, ×, or Escape |
| submitting | "Cargar" disabled with spinner until the job is accepted | Cargar |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| users | array | yes | — | The four users for the visibility checklist |
| maxSizeBytes | number | no | 10 MB | Client-side size check |
| acceptedTypes | array | no | csv, json, geojson | Client-side format check |
| onSubmit | function | yes | — | Emits {name, file, geometrySource, visibleColumns, visibility} |
| onCancel | function | yes | — | Closes after confirmation |

Validation: name required; file size and extension checked before any upload starts (AC2.1.2, AC2.1.3); step 2 requires a geometry source (AC2.2.3, AC2.4.1); out-of-range coordinate rows and geometry-less features are warnings, not blockers (AC2.2.2, AC2.3.2).

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | dialog labelled "Agregar datos"; step indicator is a list with aria-current on the active step |
| Keyboard interaction | Focus trapped; first field focused on each step; Enter advances when "Siguiente" is enabled; Escape opens the cancel confirmation |
| Errors | Inline text associated with the field through aria-describedby |
| Focus management | On close, focus returns to "Agregar datos" |

## Feature Popup

| Field | Value |
|---|---|
| Component | FeaturePopup |
| Description | Attribute card anchored to the clicked feature |
| Category | display |

States: loading (three skeleton rows), populated (visible attributes in upload order, "Sin valor" for empty), error ("No se pudieron obtener los atributos" with "Reintentar"), long (internal scroll past 8 rows). Closes on Escape, click or tap elsewhere (AC4.3.2). Role: dialog, non-modal, titled by the layer name; focus moves in on open and back to the map on close. Never overlaps the side panel; flips side when near the edge.

## Header 2D/3D Control

| Field | Value |
|---|---|
| Component | ViewModeToggle |
| Description | Two-option segmented control, 2D default |
| Category | navigation |

Role: radiogroup with radios "Vista 2D" and "Vista 3D"; arrow keys switch; the map keeps its centre on switch (AC5.1.1, AC5.1.2). While terrain loads the banner "Cargando terreno..." is shown (AC5.1.3); on terrain failure the control snaps back to 2D and a toast shows "La vista 3D no está disponible ahora" (US5.5).

## Banners and Toasts

| Component | Text | When | Dismiss |
|---|---|---|---|
| Info banner | "Vista 3D disponible para la localidad piloto" | 3D view outside the pilot area | Persistent while outside |
| Info banner | "Cargando terreno..." | terrain tiles loading | Auto when loaded |
| Toast (error) | "La vista 3D no está disponible ahora" | terrain failure | 5 s or click |
| Toast (success) | "Capa cargada" with layer name | job completed | 5 s |
| Toast (warning) | "Capa cargada con N filas con error" | job completed with failures | 5 s, link "Ver detalle" |

Toasts use aria-live="polite"; banners are part of the main region.

## Job Detail Dialog

Modal dialog (role dialog, h2 = "Carga: <layer>") with status line, timings, counts, a table of failed rows (columns "Fila", "Motivo"), original-file line, and "Cerrar". States: running (progress bar), completed, completed with errors, failed. Escape closes; focus returns to "Ver detalle".

## Visibility Dialog

Modal dialog (h2 = "Visibilidad: <layer>") with a radio group ("Todos los usuarios", "Solo estas personas") and a checkbox group of users where the owner is checked and disabled. "Guardar" applies immediately; "Cancelar" and Escape discard.

## Delete Confirmation

Modal dialog with the layer name in the text, "Cancelar" (default focus) and "Eliminar" (`btn-error`). Enter on "Eliminar" only after explicit focus, never as the default action.

## Sign-in Form

Native form with labelled email and password fields, "Iniciar sesión" submit, inline error alert. Loading state disables the button. The error never distinguishes unknown email from wrong password (AC1.1.2). Session expiry redirect shows "Tu sesión expiró. Inicia sesión de nuevo." (AC1.2.2).

## Side Panel and Drawer

Desktop: fixed 320 px panel beside the map; `«` collapses to a 24 px rail with `»` (US4.8). Tablet: daisyUI drawer over the map, opened from the header `▤`, closed by `«`, tapping the map, or swiping left; the map is inert while the drawer is open on tablet.

## Assumptions & Open Questions

- Inline style control is a proposal pending the user's decision (refined-mockups Q5). [assumption]
- Toast durations (5 s) and the 8-row popup scroll threshold are design defaults, not requirements. [assumption]
