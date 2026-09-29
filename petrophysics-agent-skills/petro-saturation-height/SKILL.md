---
name: petro-saturation-height
description: Saturation-height functions - Leverett J and Brooks-Corey fits of capillary pressure, lab-to-reservoir conversion, the free-water level, Sw from height and its comparison with log saturation. Load before petro_saturation_height.
---

# Saturation-height

## Data and fit

- **Register the data**: capillary-pressure data (`capillary_pressure`: sample, porosity, permeability_md, pc_psi / pc_bar / pc_kpa, sw, system) from mercury, porous-plate or centrifuge tests.
- **Leverett J** (`method: "leverett_j"`): J = 0.21645 Pc(psi) / (sigma cos theta) x sqrt(k/phi) normalises plugs of different quality. One curve, Sw = Swirr + a J^-b (capped at 1), is fitted to all plugs.
  - Needs `sigma_cos_lab`, the interfacial tension times the cosine of the contact angle of the lab system (e.g. air-brine about 72 dyne/cm, mercury-air about 367 dyne/cm), with source.
  - Check `rms_sw`. Above about 0.05 the plugs do not share one J curve: several rock types. Fit per rock type.
- **Brooks-Corey** (`method: "brooks_corey"`) fits each plug separately (Swirr, entry pressure, lambda). Use it to compare plugs, not to predict along a well.

## From lab to reservoir

With a well (PHIE, PERM, trajectory), the tool predicts `SW_SHF` from height above the free-water level:
- **Reservoir capillary pressure**: Pc = (rho_brine - rho_hc) g h.
- **J**: computed with the reservoir sigma cos theta and the log PHIE and PERM, then the fitted curve applied.

Parameters, each with a source:
- **`sigma_cos_res`**: reservoir interfacial tension x cos theta, e.g. gas-brine about 50, oil-brine about 26 dyne/cm, or from PVT or a report.
- **`brine_density`** and **`hydrocarbon_density`**: from pressure gradients or PVT.
- **`fwl_tvdss_m`**: the free-water level, where capillary pressure is zero, in m TVDSS. It lies below the water-saturation contact by the entry height, and often comes from pressure gradients. If unknown, run cases, and report it as the main uncertainty.

## Reading the comparison

The result gives `SW_SHF` and log `SW` per zone:
- **Agreement above the transition zone** supports the FWL, the J fit and the saturation model together.
- **SW_SHF systematically lower than log SW**: Rw too low, m or n too high, the FWL too deep, or the shale correction too small.
- **SW_SHF higher than log SW**: the opposite, or a lower-quality rock than the J plugs.
- **A mismatch confined to one zone**: a different rock type or a separate pressure compartment. Raise it as a contradiction with those alternatives.

## Reporting

- **Measurements**: the J fit (Swirr, a, b, points, RMS), and SW_SHF against log SW per zone.
- **Assumptions**: FWL, densities, sigma cos theta values with sources.
- **Claims**: the FWL is `supported` only with pressure data.
