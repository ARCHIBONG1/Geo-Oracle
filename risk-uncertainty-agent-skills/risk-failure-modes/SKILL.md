---
name: risk-failure-modes
description: Failure modes per domain from a fixed library - the mechanism, the elements, their statuses, what would detect each mode, and the register entries bearing on it - without numeric ratings. Load before ru_failure_modes.
---

# Failure modes

`ru_failure_modes` lists, for the domain and the objective, every mode in a fixed library (hydrocarbon, CO2 storage, geothermal, natural hydrogen, hydrogen storage): the mechanism, the elements involved, each element's status from the play matrix, the weakest of them, what would detect the mode, and the register entries that bear on it.

| Domain | Modes |
|---|---|
| hydrocarbon | no charge, no reservoir, seal leak, no trap, timing, not preserved |
| CO2 storage | caprock leak, fault reactivation, well leak, plume out of the complex, no injectivity |
| geothermal | insufficient permeability, cooler than modelled, induced seismicity, no recharge |
| natural hydrogen | no generation, not retained, consumed |
| hydrogen storage | containment, conversion, cycling damage |

A mode is never rated. What is said about it is which elements it involves, where they stand, and which entries bear on it; a mode whose elements stand at unknown with several entries bearing on it is where the register's attention goes, and that is a count, not a score.

Caprock, faults and wells are separate modes in storage domains because they fail and are monitored differently; folding them into one "seal" hides the one that matters.

## Reporting

Modes as `interpretations` with the elements' statuses; the product `failure_modes`.
