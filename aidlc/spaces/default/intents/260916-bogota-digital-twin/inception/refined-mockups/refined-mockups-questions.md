# Refined Mockups Questions

Context carried forward: map-first layout with a collapsible side panel, add-data wizard over the panel, popup on click, explicit 2D/3D toggle, Spanish UI, desktop and tablet, neutral styling with a standard component library, no formal accessibility target; new since the wireframes: a sign-in screen, per-layer visibility, layer deletion, wizard cancel, panel collapse, job detail with failed rows. The questions below settle the details the refined mockups and interaction specification need.

## Q1. Which component library gives the neutral styling?

The frontend is Angular; the choice fixes the look of buttons, dialogs, forms and lists.

A. Angular Material
B. PrimeNG
C. No library; hand-written components with a small CSS utility set
D. Not yet defined
X. Other (please specify)

[Answer]: X. Other — tailwind and daisyui

## Q2. How should the sign-in screen look?

A. A centred card with email, password and "Iniciar sesión", on a neutral background
B. A split screen: sign-in card on one side, a static map image of Bogotá on the other
C. Not yet defined
X. Other (please specify)

[Answer]: A. A centred card with email, password and "Iniciar sesión", on a neutral background

## Q3. Where does the uploader set a layer's visibility?

A. In a per-layer menu ("Visibilidad") that lists the four users with checkboxes, editable any time; the wizard itself does not ask
B. In wizard step 3 at upload time, and editable later from the per-layer menu
C. Not yet defined
X. Other (please specify)

[Answer]: B. In wizard step 3 at upload time, and editable later from the per-layer menu

## Q4. How are style, visibility and delete reached for a layer?

A. A three-dot menu on each layer row with "Estilo", "Visibilidad", "Eliminar"
B. Icon buttons that appear on hover or focus of the row
C. Not yet defined
X. Other (please specify)

[Answer]: A. A three-dot menu on each layer row with "Estilo", "Visibilidad", "Eliminar"

## Q5. How is the style control shown?

A. Expands inline under the layer row (colour, opacity, point size)
B. A small popover anchored to the row
C. Not yet defined
X. Other (please specify)

[Answer]: C. Not yet defined (user's words: "no defined yet"; the mockups propose inline expansion as a default, marked as an assumption)

## Q6. How does the add-data wizard appear?

A. Over the side panel, replacing the layer list while open (as wireframed)
B. As a centred modal dialog over the map
C. Not yet defined
X. Other (please specify)

[Answer]: A. Over the side panel, replacing the layer list while open (as wireframed)

## Q7. How does the job detail ("Ver detalle") appear?

A. A dialog listing the job's status, timings and the table of failed rows with reasons
B. A separate page with the same content
C. Not yet defined
X. Other (please specify)

[Answer]: A. A dialog listing the job's status, timings and the table of failed rows with reasons

## Q8. How does the layout adapt on a tablet?

A. Above 1024 px the panel sits beside the map; between 768 and 1023 px it overlays the map and can be collapsed; controls are at least 44 px
B. The panel always overlays the map on every width
C. Not yet defined
X. Other (please specify)

[Answer]: A. Above 1024 px the panel sits beside the map; between 768 and 1023 px it overlays the map and can be collapsed; controls are at least 44 px

## Consolidated Summary Confirmation

- Component library: Tailwind CSS with daisyUI for neutral styling
- Sign-in: a centred card with email, password and "Iniciar sesión"
- Visibility: set in wizard step 3 at upload time, editable later from the per-layer menu
- Layer actions: a three-dot menu per row with "Estilo", "Visibilidad", "Eliminar"
- Style control: not yet defined; the mockups propose inline expansion under the row as a default assumption
- Wizard: over the side panel, replacing the layer list while open
- Job detail: a dialog with status, timings and the failed-rows table
- Tablet: panel beside the map above 1024 px, overlay and collapsible between 768 and 1023 px, controls at least 44 px

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
