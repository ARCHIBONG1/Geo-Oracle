---
name: risk-materiality
description: The four flip tests - play status, maturity, threshold straddle, scenario comparison - and the sensitivity read; which to use for which conclusion; what the classes mean; the report and its refusal. Load before any flip test.
---

# Materiality

## Which test for which conclusion

| Conclusion | Test | Inputs | Alternatives |
|---|---|---|---|
| a play's status or capping element | `ru_flip_play` | `element_status_matrix`, the play, the element | the plausible statuses on the play scale (the ones the discriminating evidence could reach) |
| a candidate's maturity | `ru_flip_maturity` | `prospect_definition`, the readiness test the uncertainty feeds | pass, fail, not tested as the alternatives plausibly reach |
| a threshold decision (fault stability at a pressure, a column against a capacity, a closure across realizations) | `ru_straddle` | realization values or a range from a product; the threshold with its source | — |
| alternative fills, contacts or models | `ru_compare_scenarios` | `scenario_set`; the recorded outcome per scenario | — |
| a volume range | `ru_sensitivity_read` | `volume_ranges`; a geological threshold if the decision context has one | — |

Every test takes the entry id it is for, and returns a `test_record` with its provenance id: that record is what `ru_materiality_report` accepts.

## Reading the result

- A flip is a change in a categorical conclusion: the play's status or its capping element; the maturity; both sides of a threshold occupied; a different maturity or status between scenarios.
- A range that changes with nothing categorical is contributing. A volume without a geological threshold is contributing at most: there is no commercial threshold in this system.
- Every case on the failing side of a threshold is not a straddle: it is a settled failure, reported as such.
- The flip tools run the owners' rules; the base case they compute is the owner's own result, and a mismatch there is a fault to report.

## Plausible alternatives

The alternatives are the statuses or outcomes the discriminating evidence could reach, not every value on the scale: for well containment unknown, "inferred" (an integrity record) and "demonstrated" (a tested well); for a validity record untested, "pass" and "fail". Say in the findings which alternatives were tested and why.

## The report (`ru_materiality_report`)

Every entry's class from its tests. An entry with no test is undetermined and needs a reason in the findings; a material class without a provenance id is refused.

## Reporting

Flip results as `measurements` with the tool's `prov:` id; materiality as `claims` of classification inference, supported when computed.
