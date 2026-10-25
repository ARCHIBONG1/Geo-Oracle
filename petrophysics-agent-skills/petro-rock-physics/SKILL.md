---
name: petro-rock-physics
description: Rock physics and the elastic handoff. It covers fluid properties (brine, oil, gas, CO2), Vs measured or predicted, Gassmann substitution (uniform and patchy) and Backus upscaling. It also covers the elastic_logs product that the seismic specialist uses for synthetics and AVO. Load before petro_fluid_properties or petro_elastic_logs.
---

# Rock physics

## Fluids: `petro_fluid_properties`

The parameter set needs:
- `reservoir_pressure_mpa` and `reservoir_temp_c`, from `petro_pressure`, `petro_temperature` or the task;
- `salinity_ppm` (brine);
- `oil_api`, `gor_l_per_l` and `gas_gravity` (oil);
- `gas_gravity` (gas).

The methods are Batzle-Wang (1992) for brine, oil and gas, and Span-Wagner for CO2. The tool flags CO2 near its critical point (31 degC, 7.4 MPa). There, small P or T errors change its density a lot. Then run the substitution at the ends of the P and T range.

## Elastic logs: `petro_elastic_logs`

**Needs**: a well with trajectory, DT, RHOB, VSH, PHIE (and SW where hydrocarbon-bearing), and the parameter set above. The parameter set also needs `srd_elevation_m`: the seismic reference datum above MSL, with source. Agree it with the seismic specialist.

- **Vs**: `measured` (DTS) when present. Otherwise **Greenberg-Castagna**: predicted on the brine-substituted rock, then substituted back.
  - Predicted Vs carries its uncertainty into AVO: say "Vs predicted" wherever it matters.
  - Set `matrix` (sandstone, limestone, dolomite) from lithology.
- **In-situ fluid**: `in_situ_hydrocarbon` (gas, oil, CO2) fills 1 - SW. If it is wrong, every substitution is corrupt.
- **Substitution** (`substitution`: brine, gas, oil, CO2; `target_saturation`; `md_range` for the reservoir only):
  - Gassmann with the shear modulus unchanged, and the mineral modulus from matrix and VSH (Voigt-Reuss-Hill). By default, the clay in that mix is the Gulf-clay effective value (K 20.9, mu 6.85 GPa, 2.58 g/cm3: Han, Nur & Morgan 1986, extrapolated to pure clay). It is not a measured mineral value. Published clay moduli differ widely. So pass `clay_moduli` [K, mu, density] when the clay's mineralogy is known, and say which clay was used.
  - **Uniform** mixes fluids first (Wood): a little gas softens the rock almost as much as a lot. **Patchy** averages fully saturated patches (Hill), which is stiffer.
  - **For CO2 storage, always run both**: they bound the seismic response that monitoring must expect.
  - The tool leaves tight or very shaly samples (porosity below 0.03) unchanged.
- **Backus** (`backus_window_m`): upscales thin layers to the seismic scale by the full Backus (1962) average (C11, C13, C33, C44, C66). It gives the vertical velocities and Thomsen's epsilon, delta and gamma, which AVO at non-normal incidence needs. Use about a quarter of the dominant wavelength (for example 3000 m/s / 30 Hz / 4 = 25 m). Synthetic ties usually use unblocked logs, and AVO modelling of thin beds uses Backus.
- **`time_depth`**: optionally a time_depth product or table, which adds `twt_ms` to the product.

**Product**: `elastic_logs` CSV with depth_m (below the SRD), vp_m_s, vs_m_s, rhob_g_cc (+ twt_ms; + epsilon, delta, gamma when Backus averaging was applied). This is the seismic `well_logs` format. Epsilon, delta and gamma are Thomsen's parameters of the equivalent VTI medium, and vp_m_s and vs_m_s are then the vertical velocities. The product's sidecar states the fluid case, the Vs source, Backus and conditions. Publish one product per fluid case that the seismic specialist needs (in situ, brine, gas, CO2 uniform and patchy). List each as `Product: elastic_logs — <ref>`, with the case in words.

## Reporting

- **Measurements** (derived): Vp and Vs ranges per zone and the change on substitution. For example: "SAND_B gas to brine: Vp +3.6%, Vs -3.6%".
- **Assumptions**: fluid descriptors, P and T, matrix, SRD.
- **Limitations**: predicted Vs, and Gassmann's assumptions (connected porosity, low frequency, homogeneous mineral). Anisotropy is reported only when Backus averaging was applied (the layering-induced VTI). Intrinsic anisotropy is not modelled.

<!-- end of skill: petro-rock-physics -->
