---
name: petro-saturation-height
description: Saturation-height functions and seal capacity. It covers Leverett J and Brooks-Corey fits of capillary pressure, lab-to-reservoir conversion and the free-water level. It also covers Sw from height, its comparison with log saturation, and the caprock seal capacity for CO2, hydrogen or hydrocarbons from mercury-injection data. Load before petro_saturation_height.
---

# Saturation-height

## Data and fit

`petro_saturation_height` does the fits, the conversion and the comparison described here. What follows is how to choose its inputs and read its result.

- **Register the data**: capillary-pressure data from mercury, porous-plate or centrifuge tests (`capillary_pressure`). Required: sample, pc_psi / pc_bar / pc_kpa, and one saturation column. That is sw, or the mercury saturation s_hg / snw, or s_hg_pct / snw_pct in percent of pore volume. Optional: porosity, permeability_md, system, well, md_m. The tool reads the mercury saturation as sw = 1 - Snw. Porosity is optional: Leverett J needs it, and Brooks-Corey and seal capacity do not use it.
- **Systems**: the tool reads the phase words in `system`: mercury (Hg, MICP), gas (air, nitrogen, N2, methane), oil, and brine or water. The known systems are air-mercury, gas-brine (or air-brine), oil-brine and gas-oil. The tool refuses another pair, for example mercury-brine. A table that mixes systems needs one `sigma_cos_air_mercury`, `sigma_cos_gas_brine`, `sigma_cos_oil_brine` or `sigma_cos_gas_oil` per system present. Otherwise it refuses. One system uses `sigma_cos_lab`.
- **Leverett J** (`method: "leverett_j"`): J = 0.216601 Pc(psi) / (sigma cos theta) x sqrt(k/phi) normalises plugs of different quality. The tool fits one curve, Sw = Swirr + a J^-b (capped at 1), to all plugs.
  - Needs `porosity` and `permeability_md` for every sample. The tool refuses and names a sample without them. It also needs `sigma_cos_lab`, with source: the interfacial tension times the cosine of the contact angle of the lab system. For example, air-brine about 72 dyne/cm, air-mercury 480 x cos 40 = 367.7 dyne/cm (Schowalter 1979).
  - Check `rms_sw`. If it is above about 0.05, the plugs do not share one J curve: there are several rock types. Fit per rock type.
- **Brooks-Corey** (`method: "brooks_corey"`) fits each plug separately (Swirr, entry pressure, lambda). Use it to compare plugs, not to predict along a well. A plug with fewer than 4 points with Pc > 0 is not fitted (`per_plug` gives the reason).

## From lab to reservoir

With a well (PHIE, PERM, trajectory), the tool predicts `SW_SHF` from height above the free-water level:
- **Reservoir capillary pressure**: Pc = (rho_brine - rho_hc) g h.
- **J**: computed with the reservoir sigma cos theta and the log PHIE and PERM, then the fitted curve applied.
- **Gaps**: above the FWL, SW_SHF is missing where PERM, PHIE or TVDSS is missing. `sw_shf_unavailable_samples` counts them.

Parameters, each with a source:
- **`sigma_cos_res`**: reservoir interfacial tension x cos theta. For example, gas-brine about 50 and oil-brine about 26 dyne/cm. Or take it from PVT or a report.
- **`brine_density`** and **`hydrocarbon_density`**: from pressure gradients or PVT.
- **`fwl_tvdss_m`**: the free-water level, where capillary pressure is zero, in m TVDSS. It lies below the water-saturation contact by the entry height, and often comes from pressure gradients. If it is unknown, run cases, and report it as the main uncertainty.

## Reading the comparison

The result gives `SW_SHF` and log `SW` per zone:
- **Agreement above the transition zone** supports the FWL, the J fit and the saturation model together.
- **SW_SHF systematically lower than log SW**: Rw too low, m or n too high, the FWL too deep, or the shale correction too small.
- **SW_SHF higher than log SW**: the opposite, or a lower-quality rock than the J plugs.
- **A mismatch confined to one zone**: a different rock type or a separate pressure compartment. Raise it as a contradiction with those alternatives.

## Seal capacity: `method: "seal_capacity"`

The caprock's capillary seal capacity for a buoyant fluid (CO2, hydrogen, gas or oil), from mercury-injection (MICP) curves of caprock samples.
1. **Threshold pressure per sample** (measured): the capillary pressure at non-wetting saturation `threshold_snw`. The tool uses `pc_psi` as given: confirm that Pc is gauge and closure-corrected. This is important for a low threshold, because that sample can be the weakest. The default 0.10 is Schowalter's (1979) 10 % mercury saturation rule. Another pick needs a stated reason. `threshold_bound` is `value`, `lower` or `upper` when the curve does not bracket the pick.
2. **Reservoir entry pressure** (derived): Pce_res = Pce_lab x (sigma cos theta)_res / (sigma cos theta)_lab.
3. **Column height** (estimate): h = Pce_res / ((rho_brine - rho_buoyant) g). This is the height above the free-water level at which the seal leaks (Smith 1966 eq. 4).
4. **Optional reservoir threshold**: register `reservoir_threshold_psi` (low/high) for the reservoir rock below the seal. Give it in the laboratory system of the seal curves and read it with the same pick. The tool converts it with the same ratio and also gives `column_above_100pct_water_m_low_base_high`: (Pce_res - Pd_res) / ((rho_brine - rho_buoyant) g), the continuous column above the 100 % water level (Smith 1966 eq. 9). Report both columns, each with its datum. A table that mixes systems is refused with this parameter: split it by system.

**Parameters**, each with a source and low/high: `sigma_cos_lab` (or the per-system names), `sigma_cos_res`, `brine_density`. The buoyant density comes from `buoyant_fluid`:
- `co2` or `h2`: the equation of state at `reservoir_pressure_mpa` and `reservoir_temp_c`. Pure fluid only: impurities and dissolution are not modelled. A CO2 state whose density changes quickly with P or T is warned with its reason (`state_sensitive`, `sensitivity_reason`): carry it into `limitations`.
- `gas` or `oil`: `hydrocarbon_density` if registered, else Batzle-Wang (+ `gas_gravity` or `oil_api`).
- `given`: `buoyant_fluid_density` (or `hydrocarbon_density`).

**CO2-brine sigma cos theta** is an `assumption` with a wide range:
- Interfacial tension: about 25-55 mN/m (= dyne/cm) over storage conditions (Reynolds, Blunt & Krevor 2013, SCA2013-033). It falls with pressure.
- Contact angle through brine: about 27-43 degrees measured on quartz and mica at 40 degC (Mills, Riazi & Sohrabi 2011, SCA2011-06). About 70 degrees was reported on mica at 100 bar (Chiquet & Broseta 2005, as cited there). Dense CO2 can make mica and coal less water-wet (Chalbaud et al. 2007, SCA2007-09).
- Low case: the lowest tension with the highest angle. High case: the highest tension with the lowest angle. State both inputs.
- Hydrogen-brine: take the tension and angle from a cited source, as an assumption with a range.

**Read the result** (`result.seal_capacity` and the `seal_capacity` product):
- Report the weakest sample (`weakest_sample`, the lowest column) with its low, base and high column, and `sample_count`.
- Label each value by its class (`classes`): the threshold is measured, the reservoir entry pressure is derived, the column height is an estimate.
- Carry the `caveats` into `limitations`: closure, conformance and confining-stress corrections; cuttings against plugs; wettability change by CO2; diffusion and geochemistry not modelled.
- A null high column means that the density ranges allow zero contrast.
- With `reservoir_threshold_psi`: a sample with `capillary_seal` false has a column of 0 (no capillary seal). `ranges_overlap` true means that the low column is 0. Report both in `limitations`.

**Limits**:
- Low permeability is not a seal-capacity measurement.
- A caprock with logs only: compute Vsh, porosity and lithology as indicators. Put seal capacity in `missing_data` (MICP on caprock samples).
- Fracture and geomechanical seal capacity belong to structural_geology. Geochemical reaction of the seal is outside petrophysics. Route both in `recommended_followups`.
- Storage volumes are not yours: give porosity, net thickness and fluid density with ranges.
- For hydrogen, mark the result as emerging practice.

## Reporting

- **Measurements**: the J fit (Swirr, a, b, points, RMS), and SW_SHF against log SW per zone. For seal capacity, the thresholds per sample.
- **Assumptions**: FWL, densities, sigma cos theta values with sources.
- **Claims**: the FWL is `supported` only with pressure data. A seal-capacity column is an estimate: at most `partially_supported`.

<!-- end of skill: petro-saturation-height -->
