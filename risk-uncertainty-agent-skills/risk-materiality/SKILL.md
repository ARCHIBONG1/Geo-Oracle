---
name: risk-materiality
description: The four flip tests (play status, maturity, threshold straddle, scenario comparison) and the sensitivity read. Which to use for which conclusion, what the classes mean, and the report and its refusal. Load before any flip test.
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

Every test takes the entry id it is for. It returns a `test_record` with its provenance id, and `ru_materiality_report` accepts that record.

## Reading the result

- A flip is a change in a categorical conclusion:
  - the play's status or its capping element;
  - the maturity;
  - both sides of a threshold occupied;
  - a different maturity or status between scenarios.
- A range that changes with nothing categorical is contributing. A volume without a geological threshold is contributing at most: there is no commercial threshold in this system.
- If every case lies on the failing side of a threshold, that is not a straddle. It is a settled failure, class `settled_failure`. Report it beside the material entries as decisive. It takes no reduction action on its range, because no narrowing can change it. `ru_reduction_plan` lists it with its open questions, the threshold's source and the basis of the inputs. Those are what to check.
- The flip tools run the owners' rules. The base case they compute is the owner's own result. A mismatch there is a fault to report.

## Plausible alternatives

The alternatives are the statuses or outcomes that the discriminating evidence could reach, not every value on the scale. Examples:
- for well containment unknown, "inferred" (an integrity record) and "demonstrated" (a tested well);
- for a validity record untested, "pass" and "fail".

Say in the findings which alternatives were tested and why.

## The report (`ru_materiality_report`)

The report gives every entry's class from its tests. An entry with no test is undetermined, and it needs a reason in the findings. The tool refuses a material class without a provenance id. The tool draws the register with its materiality classes and one flip diagram of its tests in `result.figures`. Publish them.

## Reporting

Give flip results as `measurements` with the tool's `prov:` id. Give materiality as `claims` of classification inference, supported when computed.

<!-- end of skill: risk-materiality -->
