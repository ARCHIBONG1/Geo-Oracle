---
name: structural-geomechanics
description: Assembling the stress tensor with sourced magnitudes, slip and dilation tendency, the critical pore-pressure increase as ranges, the Andersonian check (V5), Mohr diagrams, and the dependency ceiling on stability claims. Load before sg_stress_tensor, sg_fault_stability or sg_andersonian_check.
---

# Geomechanics and fault stability

## The stress tensor: `sg_stress_tensor`

| Input | Preferred source | Fallback (flag as an assumption) |
|---|---|---|
| Sv | integrated density log (layers) from wells_petrophysics | average density 2300-2500 kg/m3 |
| Pp | pressure_profile product | hydrostatic (1030 x 9.81 x depth) |
| Shmin | leak-off or minifrac from wells_petrophysics | frictional-equilibrium bound (the tool does this and warns) |
| SHmax | breakout and fracture analysis | bounded by frictional equilibrium; run a range |
| Regime and SHmax azimuth | regional stress_field (regime counts, azimuth with its spread) | local breakouts when supplied |

The result lists the principal stresses for the regime (NF: Sv > SHmax > Shmin; SS: SHmax > Sv > Shmin; TF: SHmax > Shmin > Sv) and the frictional limits. Every input's source goes in `sources` and into `assumptions`.

## Stability: `sg_fault_stability`

- Slip tendency Ts = tau / (sigma_n - Pp) (Morris et al. 1996); dilation tendency Td = (S1 - sigma_n)/(S1 - S3) (Ferrill et al. 1999); critical pressure increase dP = (sigma_n - Pp) - (tau - C)/mu, the pore-pressure rise to Mohr-Coulomb failure.
- Ranges from a deterministic grid: friction 0.4, 0.6, 0.8 (Byerlee 1978) and the Shmin and SHmax ranges you give (from the bounds, or +/- the measurement uncertainty). The grid is reported; no random draws.
- Give `planned_pressure_increase_mpa` for injection or production questions: the verdict per fault is stable in all cases, fails in some, or fails in all. "Fails in some" is `partially_supported` with the case counts.
- Vertical faults perpendicular to SHmax carry no shear stress (a principal plane): Ts near 0 is right, not an error.
- Ts at or above 1 in some cases means that stress state is implausible for an existing fault, or the magnitudes are wrong: say which.

## The Andersonian check: `sg_andersonian_check` (V5)

Expected new-fault orientations: NF dip about 60 degrees, strike parallel to SHmax; TF dip about 30, strike perpendicular to SHmax; SS vertical, strike about 30 degrees from SHmax. A fault outside these is inherited or reactivated, which is an interpretation to state with the regional phases, not a failure of the data.

## The ceiling

A stability claim depends on the stress field's transfer status (regional prior), the pore pressure's source and the fault orientation's uncertainty. List them in `depends_on_claim_ids`; the claim's status cannot exceed the weakest. What lifts it: local breakouts (azimuth), leak-off tests (Shmin), measured pressures.

## Reporting

- `[measurement, derived]`: "F1 critical pressure increase 4.1-6.8 MPa over friction 0.4-0.8 and Shmin +/-10 % (15 cases); planned 2.5 MPa: stable in all cases", `prov:` id, product `fault_stability`.
- Mohr diagram with `sg_render(kind=mohr)`: planes drawn on the circles, failure line stated.
