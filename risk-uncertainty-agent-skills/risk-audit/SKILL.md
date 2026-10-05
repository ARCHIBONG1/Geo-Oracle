---
name: risk-audit
description: Contradictions across specialists carried with both sides, load-bearing assumptions and what rests on them, and the bias checks - single model, range narrower than its inputs, recall as evidence, transfer without status. Load before ru_contradictions, ru_assumptions or ru_bias.
---

# The audit

## Contradictions (`ru_contradictions`)

Found deterministically: a claim resting on a contradicted claim; a claim resting on a product whose ledger fails. Stated by you with both sides when two findings cannot both hold (opposite statuses on the same element from different specialists). Each carries a resolution path naming the owner and the evidence; none is resolved by picking a side.

## Assumptions (`ru_assumptions`)

From the findings' assumption lists and the claim graph: an assumption is load-bearing when the claim it is attached to has dependants, and the audit lists everything that rests on it. Hidden defaults (a parameter nobody sourced) are assumptions too: the prospect agent's `points` list and the petrophysics parameter sets are where to look.

## Bias (`ru_bias`)

| Check | What it finds |
|---|---|
| single model | an objective with one scenario where the data allow an alternative |
| range narrower than its inputs | a correlation that narrows a volume by more than a fifth; inputs with no spread |
| recall as evidence | a supported claim whose source type is unknown: nothing in the data traces it |
| transfer without status | an analogue statement carried as supported |

Each finding names the owner and the evidence. A bias finding is not an accusation; it is a place where the record does not support the confidence shown, and the owner is asked to supply what would.

## Reporting

Contradictions in `contradictions` with both claim ids; load-bearing assumptions in `assumptions` with what rests on them; bias findings as `interpretations` naming the owner.
