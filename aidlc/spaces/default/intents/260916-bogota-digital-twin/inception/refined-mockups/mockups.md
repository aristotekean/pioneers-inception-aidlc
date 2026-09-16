# Refined Mockups: Bogotá Urban Digital Twin

Upstream: `wireframes.md` and `user-flow.md` (Rough Mockups), `stories.md` (User Stories), `requirements.md` (Requirements Analysis), `team-practices.md` (Practices Discovery). Answers: `refined-mockups-questions.md`. Mid-fidelity: real Spanish labels, layout regions, states, and the story each screen serves. Styling: Tailwind CSS with daisyUI, neutral theme [Q1].

## Screen S0: Sign-in (US1.1, US1.2, US1.4)

Centred card on a neutral background [Q2]. New since the wireframes (login became a must-have).

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│              ┌──────────────────────────────┐                │
│              │  Gemelo Digital Bogotá        │                │
│              │                              │                │
│              │  Correo electrónico          │                │
│              │  [__________________________]│                │
│              │  Contraseña                  │                │
│              │  [__________________________]│                │
│              │                              │                │
│              │  [     Iniciar sesión      ] │  (btn-primary) │
│              │                              │                │
│              │  ⚠ Correo o contraseña       │  (error state) │
│              │    incorrectos               │                │
│              └──────────────────────────────┘                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

States: default; **loading** (button shows a spinner and is disabled after submit); **error** (alert-error under the button: "Correo o contraseña incorrectos"; fields keep their values); **expired session** (the card shows "Tu sesión expiró. Inicia sesión de nuevo." above the fields when redirected from an expired session).
Accessibility note: h1 "Gemelo Digital Bogotá" inside the card; main landmark is the card; keyboard entry point is the email field (autofocus); Enter submits.

## Screen S1: Main map, 2D (US4.1 to US4.8, US2.10)

Map-first with the side panel beside the map at 1024 px and above [Q8].

```
┌────────────────────────────────────────────────────────────────────────────┐
│ ▤ Gemelo Digital Bogotá            [ 2D ][ 3D ]        (avatar ▾)  Cerrar sesión │
├────────────────────┬───────────────────────────────────────────────────────┤
│ CAPAS          [«] │                                                       │
│ ┌────────────────┐ │                                                       │
│ │[✓] ● Predios   ⋮│ │                (mapa 2D)                              │
│ │    catastrales  │ │                                                       │
│ │[✓] ● Mis puntos⋮│ │           ┌──────────────────────┐                    │
│ │    de muestreo  │ │           │ Predio catastral  [×] │                    │
│ │   ▸ Estilo      │ │           │ Código   0012345678   │                    │
│ │     color [■]   │ │           │ Área (m²)  240        │                    │
│ │     opacidad ─● │ │           │ Pisos      3          │                    │
│ │     tamaño  ─●  │ │           │ Altura (m) 9.4        │                    │
│ │[ ] ● Manzanas  ⋮│ │           │ Localidad  Teusaquillo│                    │
│ │    ⟳ cargando   │ │           └──────────▼───────────┘                    │
│ │[!] ● Encuesta  ⋮│ │                                                       │
│ │  3 filas con    │ │                                                       │
│ │  error ·Ver det.│ │                                                       │
│ └────────────────┘ │                                                       │
│ [ + Agregar datos ] │                                                       │
├────────────────────┴───────────────────────────────────────────────────────┤
│ 4.6097, -74.0817 │ 1:10 000 │ Trabajos: 1 en curso                          │
└────────────────────────────────────────────────────────────────────────────┘
```

Elements:
- Header: app title, 2D/3D segmented control (`join` of two `btn`), user avatar with "Cerrar sesión" (US1.2).
- Layer row: checkbox (toggle, US4.1), colour swatch, name (wraps, never truncates, AC4.1.2), three-dot menu `⋮` opening "Estilo", "Visibilidad", "Eliminar" [Q4]; "Visibilidad" and "Eliminar" are shown only to the uploader (AC4.6.3).
- Style control: expands inline under the row [Q5, assumption]: colour (input color), opacity (range 0 to 100), point size (range 1 to 12) for point layers; changes apply live (US4.2).
- Row states: **loading** (spinner, checkbox disabled, "cargando"), **error** (`[!]` warning, "N filas con error · Ver detalle" opening the job dialog, US2.7), **zero features** ("0 elementos", AC2.10.2).
- Panel collapse `«` (US4.8): the map fills the width; a `»` handle stays at the left edge.
- Popup (US4.3): daisyUI `card` anchored to the click; scrolls internally past 8 rows; "Sin valor" for empty attributes; error state "No se pudieron obtener los atributos · Reintentar".
- Status bar (US4.4): coordinates to four decimals, scale, "Trabajos: N en curso" or "Trabajos: ninguno en curso".

States: **empty** (panel shows "Aún no hay capas. Agrega tus datos o carga el catastro oficial." with the button, US4.7); **populated**; **partial** (25+ layers scroll inside the panel; long names wrap).
Accessibility note: h1 in the header; side panel is complementary, map is main, status bar is contentinfo; keyboard entry point is the layer list; Space toggles a layer, Enter opens its menu.

## Screen S2: Add-data wizard (US2.1 to US2.6, US2.11, US4.6)

Replaces the layer list inside the panel while open [Q6]. Three steps with a progress indicator (daisyUI `steps`).

```
Paso 1 de 3 · Archivo               Paso 2 de 3 · Columnas y geometría     Paso 3 de 3 · Confirmar
┌────────────────────┐              ┌────────────────────┐                ┌────────────────────┐
│ Agregar datos  [×] │              │ Agregar datos  [×] │                │ Agregar datos  [×] │
│ ●───○───○          │              │ ●───●───○          │                │ ●───●───●          │
│ Nombre de la capa  │              │ Detectamos 8       │                │ Capa: Mis puntos   │
│ [________________] │              │ columnas · 1 240   │                │ Archivo: p.csv     │
│ Archivo (CSV o JSON│              │ filas              │                │ Filas: 1 240       │
│  hasta 10 MB)      │              │ Geometría          │                │ Geometría: lat/lon │
│ [ Elegir archivo ] │              │ (•) latitud /      │                │ Sist. de coord.:   │
│ o arrástralo aquí  │              │     longitud       │                │   WGS84 (asumido)  │
│                    │              │ ( ) columna WKT    │                │ Visible para       │
│ ⚠ El archivo supera│              │ ( ) GeoJSON        │                │ [✓] Todos          │
│   10 MB            │              │ ⚠ 3 filas fuera de │                │ [ ] Camila [ ] ... │
│                    │              │   rango            │                │                    │
│                    │              │ Columnas visibles  │                │                    │
│                    │              │ [✓] nombre         │                │                    │
│                    │              │ [✓] tipo           │                │                    │
│                    │              │ [ ] id_interno     │                │                    │
│ [Cancelar][Siguiente]│            │ [Atrás] [Siguiente]│                │ [Atrás]  [Cargar]  │
└────────────────────┘              └────────────────────┘                └────────────────────┘
```

Elements and states:
- Step 1: layer name (required; "Siguiente" disabled while empty, AC2.5.2); file picker plus drag target; the picker alone suffices on touch (AC2.1.4). Errors inline: "El archivo supera 10 MB", "Formato no admitido: usa CSV o JSON" (AC2.1.2, AC2.1.3).
- Step 2: detected column count and row count; geometry source radios pre-selected by detection (US2.2, US2.3); warnings for out-of-range rows (AC2.3.2) and features without geometry (AC2.2.2); **blocked** state when nothing is detected: no radio selected, "Siguiente" disabled, hint "Elige las columnas con la geometría" and two selects for longitude and latitude (US2.4, AC2.2.3); visible-column checkboxes (US2.5).
- Step 3: read-only summary plus visibility: "Todos" checked by default, or a per-user checklist [Q3] (AC4.6.2); "Cargar" is the only primary action.
- "Cancelar" or `[×]` or Escape at any step opens "¿Descartar la carga?" with "Descartar" and "Seguir editando" (US2.11).
- After "Cargar": the wizard closes, the row appears as loading, the status bar counts the job (AC2.6.1).

Accessibility note: the wizard is a dialog region titled by h2 "Agregar datos"; focus is trapped and lands on the first field of each step; Escape triggers the cancel confirmation.

## Screen S3: Job detail dialog (US2.6, US2.7, US6.3)

Opened from "Ver detalle" [Q7]. daisyUI `modal`.

```
┌──────────────────────────────────────────────────────┐
│ Carga: Encuesta barrios                          [×] │
│ Estado: Completada con errores                       │
│ Inicio 10:41:02 · Fin 10:41:37 · 1 240 filas         │
│ 1 237 cargadas · 3 con error                         │
│ ┌──────┬─────────────────────────────────────────┐   │
│ │ Fila │ Motivo                                  │   │
│ ├──────┼─────────────────────────────────────────┤   │
│ │ 118  │ Latitud fuera de rango (95.2)           │   │
│ │ 402  │ Longitud vacía                          │   │
│ │ 977  │ Coordenadas no numéricas                │   │
│ └──────┴─────────────────────────────────────────┘   │
│ Archivo original: encuesta.csv · 3.1 MB · 2026-09-16 │
│                                          [Cerrar]    │
└──────────────────────────────────────────────────────┘
```

States: **running** (progress bar, rows so far); **failed** (status "Falló", single reason, no table); **completed without errors** (table replaced by "Sin filas con error"). The original-file line covers US2.8.
Accessibility note: dialog titled by h2 with the job name; the failed-rows table has column headers; Escape closes.

## Screen S4: Layer visibility dialog (US4.6, US4.9)

From the three-dot menu, "Visibilidad".

```
┌──────────────────────────────────────┐
│ Visibilidad: Encuesta barrios    [×] │
│ (•) Todos los usuarios               │
│ ( ) Solo estas personas              │
│     [✓] Camila  [✓] Andrés           │
│     [ ] Laura   [ ] (yo, siempre)    │
│                    [Cancelar][Guardar]│
└──────────────────────────────────────┘
```

The uploader is always included and cannot be unticked. Saving applies immediately to the list, tiles and popups of the other users (US4.9).
Accessibility note: dialog with h2; radio group then checkbox group; Escape cancels.

## Screen S5: Delete confirmation (US4.5)

daisyUI `modal` with "¿Eliminar la capa 'Encuesta barrios'? Se borrarán sus datos y el archivo original." and buttons "Cancelar" and "Eliminar" (`btn-error`). Deletion removes the layer for all users.

## Screen S6: 3D view (US5.1 to US5.7)

Same shell as S1; the 3D control is selected.

```
├────────────────────┬───────────────────────────────────────────────────────┤
│ CAPAS          [«] │  ⓘ Vista 3D disponible para la localidad piloto        │
│ [✓] ● Terreno      │                 ▲▲▲  ▄▄ ███ ▄▄▄                        │
│ [✓] ● Edificios 3D │            ▄▄  ▲▲▲▲▲ ██ ███ ▄▄▄ ███                    │
│ [✓] ● Predios    ⋮ │         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄                  │
│ [✓] ● Mis puntos ⋮ │                                                       │
│ [ + Agregar datos ] │  "Cargando terreno..." (while tiles arrive)           │
├────────────────────┴───────────────────────────────────────────────────────┤
│ 4.6097, -74.0817 │ inclinación 45° │ Trabajos: ninguno en curso             │
```

- "Terreno" and "Edificios 3D" rows appear only in 3D and have no menu.
- Banners: "Cargando terreno..." while loading (AC5.1.3); "Vista 3D disponible para la localidad piloto" outside the pilot area, where terrain is flat and no buildings render (US5.3); fallback to 2D with the toast "La vista 3D no está disponible ahora" (US5.5).
- Parcels with no height render flat (AC5.7.2). Popups behave as in 2D (US5.4).
- Touch: one finger pans, pinch zooms, two-finger drag tilts (AC7.2.2).
Accessibility note: same landmarks as S1; the 2D/3D control is a radio group announced "Vista 2D" / "Vista 3D"; arrow keys pan, Shift plus arrows tilt.

## Tablet layout (768 to 1023 px) [Q8]

The side panel becomes an overlay drawer (daisyUI `drawer`) over the map, opened by the `▤` button in the header and closed by `«`, a tap on the map, or swipe. The wizard and dialogs use the full drawer width. All controls are at least 44 by 44 px. The 2D/3D control and "Cerrar sesión" stay in the header; the status bar collapses to coordinates and job count.

## Screen-to-Story Map

| Screen | Stories |
|---|---|
| S0 Sign-in | US1.1, US1.2, US1.4 |
| S1 Main map 2D | US2.10, US3.2, US4.1, US4.2, US4.3, US4.4, US4.7, US4.8, US7.1 |
| S2 Add-data wizard | US2.1 to US2.6, US2.11, US4.6 (step 3) |
| S3 Job detail | US2.6, US2.7, US2.8 |
| S4 Visibility dialog | US4.6, US4.9 |
| S5 Delete confirmation | US4.5 |
| S6 3D view | US5.1 to US5.7 |
| Tablet layout | US7.2, AC1.1.4, AC2.1.4 |

## Assumptions & Open Questions

- The style control expands inline under the layer row; the user left this undecided (Q5) and may replace it with a popover without changing any story. [assumption]
- The health endpoint (US6.1) and structured logs (US6.2) have no screen; they are operational, not part of the interface. [assumption]
