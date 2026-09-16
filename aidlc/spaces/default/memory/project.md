# Project-Level Rules

> Project-specific specialisation and corrections. Loaded after `org.md` and
> `team.md` as strict-additive guidance; contradictions with broader policy
> are rejected. Populated by practices-discovery and the self-learning loop.
>
> Use sparingly: most teams don't need a project layer. Reach for it
> only when this specific project needs stable, durable guidance beyond the
> team practice (for example, package-specific release checks or an additional
> regression suite for a legacy component).

## Way of Working

<!-- Project-specific specialisation. Example: -->
<!-- This monorepo requires package-scoped branch names and a package owner -->
<!-- review in addition to the team's normal merge policy. -->

## Walking Skeleton

<!-- Project-specific specialisation. Example: -->
<!-- The walking skeleton must exercise the legacy service adapter as well -->
<!-- as the new service boundary. -->

## Testing Posture

<!-- Project-specific specialisation. -->

## Change Control

<!-- Project-specific. Mode: strict or relaxed. Strict here holds for every intent and cannot be changed from chat. -->

## Deployment

<!-- Project-specific specialisation. -->

## Code Style

<!-- Project-specific specialisation. -->

## Tech Stack

<!-- Technology choices locked for this project. -->

## Decided

<!-- Decisions made in earlier stages that should not be re-asked. -->
<!-- Format: DECIDED: [decision] (Stage [slug], [date]) -->

## Scope Overrides

<!-- Custom scope rules for this project. -->

- This repository runs AI-DLC Inception only: work stops at the approval gate after Delivery Planning. Construction and Operation are not run here; the MVP is built with gentle-ai in separate repositories that consume this record as their context. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:delivery-planning:ba24f7656cb29b2e3621c1269339152169f73250bbcf533002c7ebb274dfca18 -->

## Forbidden
<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: NEVER [behavior] (affirmed [date]) -->
<!-- Example: NEVER throw exceptions across service layer boundaries (affirmed 2026-05-17) -->

- NEVER hardcode credentials, API keys, or secrets in source code; use environment injection from a secrets manager. (affirmed 2026-09-16)

## Mandated
<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: ALWAYS [behavior] (affirmed [date]) -->
<!-- Example: ALWAYS use Result<T,E> for fallible operations in service layer (affirmed 2026-05-17) -->

- ALWAYS record per-row ingestion failures and warnings on the job or datasource status; never drop a row silently. (affirmed 2026-09-16)

## Corrections

<!-- Project-specific corrections from human feedback. -->
<!-- Format: NEVER/ALWAYS [behavior] (learned [date]) -->
- When a stage question carries 5 lettered options plus Other, present A-D in the native question widget and route E (Not yet defined / None) through the built-in Other escape, saying so in the prompt; the questions file keeps the full A-E/X set. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:intent-capture:102c42e0d4b8297a21073f23f6892882b3808f644b683d84227105ae4c663fec -->
- When the user answers a stage question with a request for a comparison instead of a choice, answer the comparison in chat with verified facts, then re-present the same question before recording an answer. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:feasibility:8e266e38b961643c9e13e9237b98782a11f1315c8e8f4490af9e3a78ba37def0 -->
- When a confirmed stage answer contradicts the initial description, the newer confirmed answer supersedes the description: record it as a constraint and log the contradiction as an issue instead of re-asking the user to reconcile the document. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:feasibility:ba79618d6af513addda890d96f4235194bac5ec65168cd1bb83df5fb1ac6fa58 -->
- When a later confirmed answer removes a need recorded by an earlier stage (for example private data leaving the proof of concept), record the effect in the current stage's artifact and leave the earlier stage's artifacts untouched. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:scope-definition:11e0ad4707a60367cd47a7a0566b64df71d97fc39fff27ccadcc2d556a394cf4 -->
- Sequence the highest learning-risk unit (the raster and 3D pipeline for this project) immediately after the foundation slice, even when the user's roadmap places it in a later phase. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:scope-definition:86eac2f52244569554103e774e84d400a33bd4c6bd7fa4adcdc87c2b0aac933e -->
- In design artifacts, write user-interface labels in Spanish (the confirmed UI language) while keeping the document prose in English (the conversation language), adding an English gloss where a label is not obvious. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:rough-mockups:c886838b752c0b2aaad401629d643fd85647908e30a76b15912023b5c9962d35 -->
- When a stage answer is vague (for example 'I will add better auth'), ask one targeted follow-up offering concrete shapes and record the precise answer; never guess the shape and never treat the vague answer as settled. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:practices-discovery:3df263124e97f1a5886a213c64e3db0a5fa3ec911db3ebcac8280a7fd0a16af7 -->
- When the user answers a performance question with 'best effort', record it as a measure-and-report requirement (which metric, under which load) rather than an untestable threshold, so the requirement still has a pass/fail check. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:requirements-analysis:768a22ec0ee873337240b84abf202da2b0e4a3e8aa1fbf5999865737d1bedc80 -->
- When the user leaves a small design detail undecided, propose a default in the artifact and mark it as an assumption the user can overrule, instead of blocking the stage on it. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:refined-mockups:ad569369a18ab53e3b0ea94574d564eff7eabb7eb657f2b1dbb98a4fc8e72174 -->
- When two confirmed answers together change a count or shape (for example four blocks plus a separately chosen block), record the implied result in the artifact and state it explicitly in the consolidated summary rather than forcing the earlier count. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:domain-design:223fea3b7497717785e55cca0699c739d534300ef3057cc3f025bd26be964f98 -->
- Present a plan-approval question in chat with a logged decision/answer pair; never add an extra H2 section to the questions file after the consolidated summary confirmation, because the confirmation receipt refuses it. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:units-generation:e58d5fab181cec282176ab0dd37a6632abb0ccc40dd5699d5d4b680d9bb2d7ac -->
- When the user answers a design question with 'use the simplest way', choose the simplest option, state the interpretation explicitly in the consolidated summary the user confirms, and record how the more complete option can be swapped in later. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:contract-design:f89f45f8b347a5221f77119261de58aa1dab72c8221289c48592ee686b79278a -->
- When the walking skeleton spans several units of work, bundle the small foundation units in full and enumerate thin slices of the larger ones in a slice table in bolt-plan.md, naming the Bolt that completes each slice. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:delivery-planning:5349a43af1dc3e404f369c0435168a1b8a56c232c5b31c8da0edd88809bdf086 -->
- When bundling units into Bolts yields fewer Bolts than the question's option text estimated, keep the coherent bundles, state the count difference in the completion summary, and name which Bolt to split if it overruns. (learned 2026-09-16) <!-- cid:260916-bogota-digital-twin:delivery-planning:108615b3dcbfc173f9c13e123d5b957c39e99a2b3c874903f7dff7e4bd51519b -->
