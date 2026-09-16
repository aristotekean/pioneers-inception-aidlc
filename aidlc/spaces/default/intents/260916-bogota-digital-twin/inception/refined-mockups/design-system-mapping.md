# Design System Mapping: Bogotá Urban Digital Twin

Upstream: `mockups.md`, `interaction-spec.md`, `wireframes.md`, `user-flow.md`, `requirements.md`, `stories.md`, `team-practices.md` (Angular with strict mode, Prettier and ESLint). Library: Tailwind CSS with daisyUI, neutral theme [Q1]; no brand guidelines apply.

## Theme and Tokens

| Token | Value | Use |
|---|---|---|
| daisyUI theme | `corporate` (light) as the single theme | Neutral styling, no dark mode in the proof of concept |
| Primary | theme `primary` | "Iniciar sesión", "Agregar datos", "Cargar", "Guardar" |
| Error | theme `error` | "Eliminar", failed-job states, error alerts |
| Warning | theme `warning` | Rows with errors, out-of-range warnings |
| Info | theme `info` | 3D banners |
| Spacing scale | Tailwind 1, 2, 3, 4, 6, 8 (4 px base) | Panel padding 4, row padding 2, dialog padding 6 |
| Type scale | `text-sm` body, `text-base` labels, `text-lg` dialog titles, `text-xl` app title | Readable at 100 percent zoom on tablet |
| Radius | `rounded-lg` cards and dialogs, `rounded` inputs | Consistent softness |

## Component Mapping

| Screen element | daisyUI / Tailwind component | Notes |
|---|---|---|
| App header | `navbar` | Title left, `join` toggle centre, `dropdown` avatar right |
| 2D/3D control | `join` with two `btn btn-sm` acting as radios | `btn-active` on the selected view |
| Side panel (desktop) | `aside` with `w-80`, `overflow-y-auto` | Collapse to `w-6` rail |
| Side panel (tablet) | `drawer` with `drawer-side` overlay | Opened from `btn btn-ghost` in navbar |
| Layer list | `ul menu` | One `li` per LayerRow |
| Layer checkbox | `checkbox checkbox-sm` | Native input, label is the name |
| Colour swatch | `w-3 h-3 rounded-full border` | Inline style from layer colour |
| Actions menu | `dropdown dropdown-end` with `menu` | "Estilo", "Visibilidad", "Eliminar" |
| Style control | `collapse` under the row with `input type=color`, `range range-xs` | Inline (assumption) |
| Add-data button | `btn btn-primary btn-block` | Always at the panel bottom |
| Wizard steps | `steps steps-horizontal` | Three steps |
| File picker | `file-input file-input-bordered` plus a dashed drop zone | Drop zone hidden on touch |
| Text inputs | `input input-bordered` with `label` | Labels above inputs |
| Radios and checkboxes | `radio`, `checkbox` | Geometry source, visible columns, visibility |
| Inline errors and warnings | `alert alert-error`, `alert alert-warning` `text-sm` | Associated to the field |
| Feature popup | `card card-compact shadow` positioned by the map adapter | Anchored, flips near edges |
| Dialogs (job detail, visibility, delete, cancel) | `modal` with `modal-box` | Native dialog element for focus trapping |
| Failed-rows table | `table table-sm table-zebra` | Columns "Fila", "Motivo" |
| Progress | `progress progress-primary` | Running job in the dialog |
| Spinner | `loading loading-spinner loading-sm` | Row loading, button loading |
| Toasts | `toast toast-end` with `alert` | aria-live polite |
| Banners | `alert alert-info` at the top of the map region | 3D messages |
| Status bar | `footer` with `text-xs` | Coordinates, scale, jobs |
| Sign-in card | `card w-96 shadow` centred with `hero` background | S0 |

## Map Engine Boundary

The map canvas (2D and 3D) is rendered by the map engine behind the frontend's map adapter, as the initial description proposes; daisyUI styles the shell around it. Popups are DOM elements positioned by the adapter, so they use the same components as the rest of the interface. The 3D engine's default widgets (timeline, animation, base-layer picker, geocoder) are disabled; the header control is the only view switch.

## Iconography

Lucide icons (open license) via inline SVG: layers, plus, more-vertical, chevrons, alert-triangle, x, eye, trash, map, cube. Every icon-only button carries an aria-label in Spanish.

## Responsive Rules

| Breakpoint | Rule |
|---|---|
| 768 to 1023 px | Panel is a drawer; controls at least 44 by 44 px; status bar shows coordinates and jobs only |
| 1024 px and above | Panel beside the map at 320 px; hover reveals row menus |
| Below 768 px | Not supported in the proof of concept; a notice recommends a larger screen |

## Assumptions & Open Questions

- The `corporate` daisyUI theme is a placeholder for "neutral"; any light theme without brand colours satisfies the answers. [assumption]
- Lucide is proposed for icons; no icon set was asked or answered. [assumption]
