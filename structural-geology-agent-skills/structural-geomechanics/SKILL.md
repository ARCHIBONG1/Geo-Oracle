---
name: structural-geomechanics
description: How to assemble the stress tensor with sourced magnitudes, slip and dilation tendency, the critical pore-pressure increase as ranges, the Andersonian check (V5), Mohr diagrams, and the dependency ceiling on stability claims. Load before sg_stress_tensor, sg_fault_stability or sg_andersonian_check.
---

# Geomechanics and fault stability

## The stress tensor: `sg_stress_tensor`

**The depth, by its datum.** Give one of these, never both:
- `depth_tvdss_m`: below sea level, the depth of horizons, wells and the `pressure_profile`;
- `depth_below_seabed_m`: below the sediment surface. Offshore, the tool adds `water_depth_m`. Onshore, it subtracts `ground_elevation_m`.

Put a target stated in TVDSS in `depth_tvdss_m` as it stands. If you pass it as depth below the seabed, it lands a water depth too deep, possibly outside the pressure profile.

**Keep the pressure_profile in the call.** The tool reads its LOT, XLOT or minifrac for Shmin, even when `pp_mpa` is given and when the depth lies outside the profile's pressures. Answer a refusal about the profile's range with `pp_mpa` plus the profile, never with `pp_mpa` instead of it. A retry without the profile loses the leak-off, and Shmin falls back to the regime's default bound. The result's `shmin_basis` says where Shmin came from. If Shmin falls to a default bound while you hold a profile with a test in it, the result lists that profile in `unused_stress_tests`. The process audit flags it.

| Input | Preferred source | Fallback (flag as an assumption) |
|---|---|---|
| Sv | integrated density log (layers) from wells_petrophysics | average density 2300-2500 kg/m3 |
| Pp | pressure_profile product | hydrostatic (1030 x 9.81 x depth) |
| Shmin | the petrophysics `pressure_profile`: a LOT, XLOT or minifrac in it (status `stress test: ...`) is read as Shmin and carried to the tensor depth at a constant effective stress ratio K = (S - Pp)/(Sv - Pp); or `shmin_mpa` with its basis | NF: frictional-equilibrium bound; SS: midway between Pp and Sv; TF: midway between Sv and SHmax (the tool warns: run the range) |
| SHmax | breakout and fracture analysis | bounded by frictional equilibrium; run a range |
| Regime and SHmax azimuth | regional stress_field (regime counts, azimuth with its spread) | local breakouts when supplied |

The result lists the principal stresses for the regime (NF: Sv > SHmax > Shmin; SS: SHmax > Sv > Shmin; TF: SHmax > Shmin > Sv) and the frictional limits. Put every input's source in `sources` and into `assumptions`.

## Stability: `sg_fault_stability`

- The tool computes:
  - slip tendency Ts = tau / (sigma_n - Pp) (Morris et al. 1996);
  - dilation tendency Td = (S1 - sigma_n)/(S1 - S3) (Ferrill et al. 1999);
  - critical pressure increase dP = (sigma_n - Pp) - (tau - C)/mu, the pore-pressure rise to Mohr-Coulomb failure.
- The ranges come from a deterministic grid: friction 0.4, 0.6, 0.85 by default, and the Shmin and SHmax ranges you give (from the bounds, or +/- the measurement uncertainty).
  - Byerlee (1978): 0.85 below 200 MPa normal stress, 0.6 above. 0.4 stands for weak clay-rich gouge below Byerlee's range.
  - The base case uses the median of the list (`base_mu`).
  - The result gives the grid and the true case count. There are no random draws.
- For injection or production questions, give `planned_pressure_increase_mpa`. The verdict per fault is stable in all cases, fails in some, or fails in all, with `fraction_of_cases_stable`. "Fails in some" is `partially_supported` with that fraction.
- Vertical faults perpendicular to SHmax carry no shear stress (a principal plane). Ts near 0 is right, not an error.
- Ts at or above 1 in some cases means one of two things: that stress state is implausible for an existing fault, or the magnitudes are wrong. Say which.

## The Andersonian check: `sg_andersonian_check` (V5)

Expected new-fault orientations:
- NF: dip about 60 degrees, strike parallel to SHmax;
- TF: dip about 30, strike perpendicular to SHmax;
- SS: vertical, strike about 30 degrees from SHmax.

A fault outside these is inherited or reactivated. That is an interpretation to state with the regional phases, not a failure of the data.

## The ceiling

A stability claim depends on the stress field's transfer status (regional prior), the pore pressure's source and the fault orientation's uncertainty. List them in `depends_on_claim_ids`. The claim's status cannot exceed the weakest. What lifts it: local breakouts (azimuth), leak-off tests (Shmin), measured pressures.

## Reporting

- `[measurement, derived]`: "F1 critical pressure increase 4.1-6.8 MPa over friction 0.4-0.85 and Shmin +/-10 % (15 cases); planned 2.5 MPa: stable in all cases", `prov:` id, product `fault_stability`.
- `sg_fault_stability` draws the Mohr diagram of the base case itself (`result.figures`): planes drawn on the circles, failure line stated.
- For another friction or set of planes, use `sg_render(kind=mohr)`.

<!-- end of skill: structural-geomechanics -->
