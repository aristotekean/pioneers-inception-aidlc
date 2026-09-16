# Discovered Rules: Bogotá Urban Digital Twin

Status: **affirmed** at the practices-discovery interview on 2026-09-16.
Only human-stated hard constraints are recorded here; every other proposed
constraint from the lead draft or the support-agent contributions was either
not put to a vote, or was put to a vote and not selected, and therefore
remains an ordinary practice or scope note in `team-practices.md` /
`evidence.md` rather than a Mandated or Forbidden rule. See `evidence.md`
for the full Q8 disposition.

## Mandated

- ALWAYS record per-row ingestion failures and warnings on the job or
  datasource status; never drop a row silently. *(Affirmed at the interview
  on 2026-09-16, Q9 — the human selected the hard-rule option after the
  developer agent flagged this as a gap between the architecture document's
  own ingestion step 7 and the draft's Mandated list.)*

## Forbidden

- NEVER hardcode credentials, API keys, or secrets in source code; use
  environment injection from a secrets manager. *(Affirmed at the interview
  on 2026-09-16, Q8 — the only option the human selected out of the five
  candidates presented; options A, B, D, and E were not selected and are
  not recorded as hard rules.)*
