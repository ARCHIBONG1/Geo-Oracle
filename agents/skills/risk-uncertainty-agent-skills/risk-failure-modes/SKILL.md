---
name: risk-failure-modes
description: Failure modes per domain from a fixed library, without numeric ratings. Each mode has the mechanism, the elements, their statuses, what would detect each mode, and the register entries bearing on it. Load before ru_failure_modes.
---

# Failure modes

When the matrix holds more than one play, give `play`: the tool refuses a matrix of several plays without it. For the domain and the objective, `ru_failure_modes` lists every mode in a fixed library (hydrocarbon, CO2 storage, geothermal, natural hydrogen, hydrogen storage). For each mode, it gives:
- the mechanism;
- the elements involved, and each element's status from the play matrix;
- the weakest of them;
- what would detect the mode;
- the register entries that bear on it.

| Domain | Modes |
|---|---|
| hydrocarbon | no charge, no reservoir, seal leak, no trap, timing, not preserved |
| CO2 storage | caprock leak, fault reactivation, well leak, plume out of the complex, no injectivity |
| geothermal | insufficient permeability, cooler than modelled, induced seismicity, no recharge |
| natural hydrogen | no generation, not retained, consumed |
| hydrogen storage | containment, conversion, cycling damage |

A mode is never rated. You say which elements it involves, where they stand, and which entries bear on it. The register's attention goes to a mode whose elements stand at unknown with several entries bearing on it. That is a count, not a score.

In CO2 storage, caprock, faults and wells are separate modes, because they fail and are monitored differently. If you fold them into one "seal", you hide the one that matters. In hydrogen storage, the library holds one `containment` mode over three elements: caprock_containment, fault_containment and well_containment. With the play matrix, the tool gives each element's status in `element_statuses`, and the weakest in `weakest_element_status`. Report the three statuses one by one, and name the element at the weakest status. Do not add modes that the tool did not return.

## Reporting

Give modes as `interpretations` with the elements' statuses, and the product `failure_modes`.

<!-- end of skill: risk-failure-modes -->
