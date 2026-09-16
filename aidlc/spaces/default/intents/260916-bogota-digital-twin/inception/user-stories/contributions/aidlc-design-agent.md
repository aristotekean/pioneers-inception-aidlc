**Collaborator:** aidlc-design-agent

## Contribution

Persona fidelity is good: Camila, Andrés and Laura stay consistent with the
confirmed institutional profiles (personas.md), their goals map cleanly to
the stories they narrate, and the priority ranking (Andrés > Camila > Laura)
matches which persona's flow each success metric depends on. Most stories
also reuse the wireframes' exact Spanish strings as acceptance-criteria
text (`"Aún no hay capas"`, `"Trabajos: 2 en curso"`, `"La vista 3D no está
disponible ahora"`), which is the right pattern — it keeps a criterion
verifiable against the literal label a researcher will see, not a paraphrase.
The gaps below are places where that pattern was not applied, or where a
wireframe interaction, screen state, or touch behaviour has no story or
criterion at all.

### 1. No story covers leaving the wizard once opened

Screen 3's accessibility note says "Escape asks to discard," but that is the
only place a cancel/dismiss path is even mentioned, and it is keyboard-only.
Laura's device is a tablet with no physical Escape key (personas.md,
wireframes.md Q6), and none of US2.1–US2.10 gives any persona a way out of
the three-step wizard without finishing it. A researcher who opens "Agregar
datos" by mistake, or wants to abandon a bad file mid-wizard, is currently
trapped — that is a worse experience than the error states the stories do
cover. Propose adding, in Group 2:

> **US2.11 Cancel the wizard**
> As Camila, I want to close the add-data wizard at any step without
> loading anything, so that opening it by mistake or changing my mind
> costs me nothing.
> Priority: Must Have. Traces: FR2.1 (wizard flow). Depends on: US2.1.
> - AC2.11.1 Given any wizard step, when I select the close control, then
>   the wizard closes, no layer is created, and I return to the main map
>   view unchanged.
> - AC2.11.2 Given a tablet viewport, when I tap the close control, then
>   the same result occurs without needing a keyboard.

### 2. No story covers collapsing the side panel

The layer panel's `[<]` collapse control appears in every map wireframe
(Screens 1 and 4) and is Flow 4's fourth step ("collapses the side panel ->
the map takes the full width"), but no story in Group 4 exercises it.
Propose adding:

> **US4.8 Collapse and expand the side panel**
> As Laura, I want to collapse the layer panel to see the full map and
> expand it again, so that I can free up screen space during a demo.
> Priority: Should Have. Traces: FR4.1 (panel), wireframes Screen 1/4.
> Depends on: US4.1.
> - AC4.8.1 Given the panel is expanded, when I select the collapse
>   control, then the panel hides and the map canvas takes the full width.
> - AC4.8.2 Given the panel is collapsed, when I select the expand
>   control, then the panel returns with its previous layer states intact.

### 3. Wireframes' "partial" layer-list state has no acceptance criteria

Screen 1's states section calls out three scannability cases the panel must
survive: 20+ layers scrolling inside the panel, long layer names wrapping
to two lines, and a zero-feature layer showing "0 elementos". None of these
appear under US4.1. This is exactly the "design for the worst case" gap —
a long researcher session will produce more than a handful of layers.
Propose adding to US4.1:
- AC4.1.2 Given more than 20 visible layers, when the panel renders, then
  the list scrolls inside the panel and the rest of the screen is unaffected.
- AC4.1.3 Given a layer with zero features, when it appears in the list,
  then it shows "0 elementos" beside its name rather than looking identical
  to a populated layer.

### 4. Empty-state text is trimmed from the wireframe's actual copy

AC4.7.1 quotes `"Aún no hay capas"` but drops the wireframe's second clause
that tells the researcher there are two ways to fill the map: `"Aún no hay
capas. Agrega tus datos o carga el catastro oficial."` Since Andrés owns
the cadastre-loading job (US3.1) and Camila owns the upload (US2.1), the
full empty-state copy is what tells a first-time visitor both paths exist.
Propose:
- AC4.7.1 (replace) Given no visible layers, when the panel renders, then
  it shows "Aún no hay capas. Agrega tus datos o carga el catastro oficial."
  with the "Agregar datos" action.

### 5. Popup close-on-touch is not stated

AC4.3.2 tests "Escape or click elsewhere" closes the popup. On Laura's
tablet there is no Escape key, and "click elsewhere" needs an explicit touch
equivalent (tap outside the popup) or a QA engineer will only test the mouse
path. Propose:
- AC4.3.2 (extend) ... add: "Given the popup is open on a touch device, when
  I tap outside the popup, then it closes the same way."

### 6. 3D progressive-loading banner has no criterion

Screen 4's states list a distinct loading banner, "Cargando terreno...",
shown while terrain tiles resolve — separate from the already-covered
fallback message (US5.5) and the pilot-boundary banner (US5.3). Propose
adding to US5.1 or US5.2:
- AC5.1.3 Given the 3D view is opening, when terrain tiles are still
  resolving, then the banner reads "Cargando terreno..." until they finish
  or the fallback in US5.5 triggers.

### 7. Status bar's zero-jobs text is untested

AC4.4.2 only covers two running jobs. The default/idle state shown in every
other wireframe screen ("Trabajos: ninguno en curso") never appears in an
acceptance criterion, so a regression that left a stale job count on-screen
between loads would not be caught. Propose:
- AC4.4.3 Given no jobs are running, when I look at the status bar, then it
  shows "Trabajos: ninguno en curso".

### 8. Sign-in screen (new, not in the wireframes) is under-specified relative to the rest of the document

FR1 authentication postdates the wireframes and user-flow document — both
say "There is no login for the proof of concept," which the requirements
explicitly supersede. That means US1.1–US1.3 are the only description this
screen has anywhere in the record, yet they are the one story group in the
document that does not quote a single exact Spanish UI string, unlike every
other group. Concretely missing:
- The sign-in screen's field labels, submit button label, and the wrong-password
  message text (AC1.1.2 only says "a message that does not reveal whether the
  email exists" — no Spanish wording to hold implementation to).
- The `"Cerrar sesión"` control's location (US1.2 quotes the label but not
  where it lives, e.g. the top bar next to "? Ayuda" per the wireframe's IA).
- Whether the sign-in screen and its controls meet the same 44×44px touch
  target named in US7.2 (AC7.2.1 is scoped to "the map, panel and controls"
  and does not name the sign-in screen, which is the one screen a tablet
  user must clear before reaching anything US7.2 does cover).
Recommend the lead either add Spanish label placeholders to US1.1–US1.3 now
(consistent with every other group) or explicitly flag the sign-in screen as
pending visual design at refined-mockups, so it is not silently the one
under-specified screen in an otherwise consistent document.

### 9. Wizard's drag-and-drop affordance has no touch fallback story

Step 1 offers "Elegir CSV o JSON... o arrastrar aquí" (choose file, or drag
here). Drag-and-drop from outside the browser is not a touch gesture a
tablet user can perform at all — Laura's device. US2.1 only tests "choose"
via file size/format, never distinguishing the two entry paths. Recommend
adding a note to AC2.1.1 (or a new AC) confirming the file-picker ("Elegir")
path alone is sufficient to complete the story on a touch device, so the
drag affordance is understood as a desktop enhancement, not the only path.

## Positions

- OBJECT: no story lets a researcher leave the add-data wizard once opened,
  and the one documented dismiss path (Escape) does not work on Laura's
  tablet — this is a real "trapped user" risk, not a nice-to-have. See
  Contribution item 1.
- OBJECT: the side panel's collapse/expand control is in every map
  wireframe and is Flow 4 step 4, but has no story — a documented
  interaction with zero test coverage. See Contribution item 2.
- OBJECT: the popup's touch-close behaviour and the wizard's touch-cancel
  behaviour are both left to the Escape key alone, which does not exist on
  the tablet persona's device; AC4.3.2 needs an explicit touch case. See
  Contribution items 1 and 5.
- OBJECT: the sign-in screen is functionally sound (AC1.1.1–AC1.3.2 cover
  happy path, wrong password, and forced redirect correctly) but is the
  only story group with no exact Spanish label text and no explicit tablet
  touch-target coverage, inconsistent with the rest of the document. See
  Contribution item 8.
- AGREE: the persona set and priority ranking are faithful to the confirmed
  institutional profiles and to which success metric each persona's flow
  proves out.
- AGREE: the wizard, popup, layer-panel, and 2D/3D-toggle stories map
  cleanly onto their wireframe screens and flows, including the load/error/
  partial states the wireframes call out for uploads (US2.6, US2.7, US2.9).
- AGREE: reusing the wireframes' exact Spanish strings inside acceptance
  criteria (where it was done) is the right pattern and should extend to
  every screen state, per Contribution items 4, 6, 7, 8.
