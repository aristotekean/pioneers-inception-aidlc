# Accessibility Checklist: Bogotá Urban Digital Twin

Upstream: `mockups.md`, `interaction-spec.md`, `requirements.md` (NFR8: no formal conformance target for the proof of concept), `stories.md`, `wireframes.md`, `user-flow.md`, `team-practices.md`. Because no formal target applies, this checklist records the baseline the mockups already meet and what is deliberately deferred, so the choice is explicit rather than accidental.

## Baseline Kept in the Proof of Concept

| Item | Where | Status |
|---|---|---|
| Keyboard reachability of every control (Tab order follows layout; Space toggles, Enter activates, Escape closes) | All screens | In mockups |
| Visible focus indicator (daisyUI default outline, 2 px) | All screens | In mockups |
| Landmarks: header, complementary panel, main map, contentinfo status bar | S1, S6 | In mockups |
| One h1 per screen, h2 for dialogs | All screens | In mockups |
| Form labels above inputs, errors associated with fields | S0, S2 | In mockups |
| Errors identified in text, never colour alone (icon plus text) | S1 rows, S2, S3 | In mockups |
| Dialogs trap focus, close on Escape, return focus to the trigger | S2, S3, S4, S5 | In mockups |
| Touch targets at least 44 by 44 px on tablet | Tablet layout | In mockups |
| Text contrast 4.5:1 with the neutral theme | All screens | To verify at build with the chosen theme |
| Icon-only buttons carry Spanish aria-labels | Header, rows | In mockups |
| Status changes announced (toasts aria-live polite) | Toasts | In mockups |
| Page language declared as Spanish | Document | To implement |

## Deliberately Deferred (no formal target, NFR8)

| Item | Reason | Revisit when |
|---|---|---|
| Full keyboard control of the map and 3D camera (arrow pan, Shift tilt) | Engine-dependent effort; noted in mockups as intended | Access control opens the tool to more users |
| Screen-reader description of map content (features, layers on the map) | Not feasible at proof-of-concept scale | Same |
| Reduced-motion preference for 3D transitions | Engine setting to confirm | Same |
| Colour-blind-safe default palette for layers | Users pick colours; defaults are neutral | Refined styling |
| Automated accessibility scan in CI | Not part of the affirmed CI gates | Build and Test if time allows |

## Per-Screen Notes

| Screen | Heading | Landmarks | Keyboard entry point |
|---|---|---|---|
| S0 Sign-in | h1 in card | main | Email field (autofocus) |
| S1 Main map 2D | h1 header | header, complementary, main, contentinfo | Layer list |
| S2 Wizard | h2 "Agregar datos" | dialog inside complementary | First field of the step |
| S3 Job detail | h2 "Carga: name" | dialog | "Cerrar" |
| S4 Visibility | h2 "Visibilidad: name" | dialog | First radio |
| S5 Delete | h2 question | dialog | "Cancelar" |
| S6 3D view | h1 header | as S1 | View toggle |

## Assumptions & Open Questions

- Contrast of the chosen neutral theme is verified at build time; if it fails 4.5:1 the theme is adjusted, not the checklist. [assumption]
