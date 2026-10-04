---
name: prospect-scenarios
description: Building trap, fill and contact alternatives from the upstream alternatives, naming what would separate them, and why they are never blended. Load before pt_scenarios.
---

# Scenarios

## Where they come from

| Kind | Source |
|---|---|
| trap | the structural product's four-way against fault-bounded closure; an alternative fault interpretation |
| fill | fill to spill against a part-filled case from a column-height range or a seal capacity |
| contact | a contact measured in a well against one inferred; a gas cap above an oil leg |
| correlation | the stratigraphic `correlation_alternatives`: a different top changes the closure's reservoir |

Each scenario cites the upstream claim or product that proposes it. A scenario you invented is not a scenario; it is a hypothesis without a basis, and the tool refuses it.

## Discriminating evidence

Every scenario names what would separate it from the others: a contact from a well in the closure, a DHI, a pressure measurement, a seal capillary-entry pressure. This is what `risk_uncertainty` reads to say which uncertainty is reducible, so a scenario without it is incomplete work.

## Never blended

Volumes are reported per scenario. Combining them into one distribution needs weights; a weight is a probability; this specialist reports no probabilities. If asked for a single number across scenarios, decline and give the per-scenario ranges with the evidence that would resolve the split.

## Reporting

Scenarios are `hypotheses` with `source_type` derived. The `scenario_set` product carries them with their bases and discriminating evidence.
