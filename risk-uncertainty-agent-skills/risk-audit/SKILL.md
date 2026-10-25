---
name: risk-audit
description: Contradictions across specialists, carried with both sides, and load-bearing assumptions and what rests on them. Also the bias checks (single model, range narrower than its inputs, recall as evidence, transfer without status). Load before ru_contradictions, ru_assumptions or ru_bias.
---

# The audit

## Contradictions (`ru_contradictions`)

The tool finds two kinds deterministically: a claim resting on a contradicted claim, and a claim resting on a product whose ledger fails. If two findings cannot both hold (opposite statuses on the same element from different specialists), you state the contradiction with both sides. Each contradiction carries a resolution path that names the owner and the evidence. Never resolve one by picking a side.

## Assumptions (`ru_assumptions`)

The audit works from the findings' assumption lists and the claim graph. An assumption is load-bearing when the claim it is attached to has dependants. The audit lists everything that rests on it. Hidden defaults (a parameter nobody sourced) are assumptions too. Look for them in the prospect agent's `points` list and the petrophysics parameter sets.

## Bias (`ru_bias`)

| Check | What it finds |
|---|---|
| single model | an objective with one scenario where the data allow an alternative |
| range narrower than its inputs | a correlation that narrows a volume by more than a fifth; inputs with no spread |
| recall as evidence | a supported claim whose source type is unknown: nothing in the data traces it |
| transfer without status | an analogue statement carried as supported |

Each finding names the owner and the evidence. A bias finding is not an accusation. It is a place where the record does not support the confidence shown. The owner is asked to supply what would.

## Reporting

Put contradictions in `contradictions` with both claim ids, and load-bearing assumptions in `assumptions` with what rests on them. Give bias findings as `interpretations` that name the owner.

<!-- end of skill: risk-audit -->
