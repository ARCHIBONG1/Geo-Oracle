---
name: risk-register
description: How to collect every uncertainty from the ledger, the forwarded findings and the product sidecars. How to type and trace each one, merge duplicates, and build the claim graph. Load before ru_collect.
---

# The register

## Collect (`ru_collect`)

Use all of these sources, every time:
- the ledger: its uncertainties, contradictions, unsettled hypotheses and open questions;
- the forwarded findings, copied exactly from the task with their ids, classifications, statuses, scope tags, `depends_on`, assumptions and `kind` (unknown, assumption, limitation, missing_data, contradiction, alternative). A claim of status unresolved, proposed or contradicted, or of classification unknown or hypothesis, is an entry;
- the products: every failing or untested ledger entry, every warning, every transfer below demonstrated.

Give the objectives by name. An entry naming two or more is shared.

## Classify (`ru_classify`)

The type follows deterministically from the source:
- a tag `[regional]` or `[analogue]` is transfer;
- a missing or untested item is data;
- an alternative is scenario (fill, contact, phase) or conceptual (which model);
- an assumption with a value is parameter;
- an entry whose affects list is non-empty and nothing else fits is dependency.

Overrides are recorded with their note. Aleatory needs a reason.

## Merge (`ru_merge`)

Duplicates (the same uncertainty raised by two specialists, or by the ledger and a product) become one entry with every source and every objective. The register product carries U1-U3. The tool draws the register in `result.figures`. Publish it.

## The claim graph (`ru_dependencies`)

The tool builds the graph from `depends_on_claim_ids` in the findings and `depends_on` in the sidecars. It gives:
- the dependants of each claim;
- the load-bearing claims (most dependants);
- shared nodes (feeding two or more objectives);
- cycles (a fault to report);
- stale nodes (resting on a superseded task).

Give the objectives as `{objective: [claim ids]}`. The tool draws the claim graph in `result.figures`. Publish it.

## Reporting

Give entries as `interpretations` in the statement form. Give the graph's load-bearing and shared claims as `interpretations` with their counts.

<!-- end of skill: risk-register -->
