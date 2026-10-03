---
name: structural-fault-seal
description: Juxtaposition, shale gouge ratio, shale smear factor and calibrated column heights from a petrophysics Vsh log and a sourced throw; thresholds, calibrations and what the seal conclusion depends on. Load before sg_fault_seal.
---

# Fault seal

## Inputs

- A `vshale_log` product from wells_petrophysics at a well near the fault, read with `sg_read_product` (method, endpoints, depth columns). Use `tvdss_m`.
- A throw at the interval of interest, with its source: a seismic throw measurement, a throw profile (phase 1), or a stated offset; cite it in `depends_on`.
- The interval on the fault (top and base) must lie inside the log, and the log must reach `top - throw`: the tool refuses otherwise and tells you the log's range.

## What `sg_fault_seal` computes

- **Juxtaposition**: at each point on the fault, the footwall lithology against the hanging-wall lithology displaced by the throw, classed sand or shale by the Vsh cutoff (0.35 default; state it). Sand-sand windows are where juxtaposition alone does not seal.
- **SGR** (Yielding et al. 1997): the Vsh-weighted thickness of the beds that slipped past each point, over the throw, in percent. The gouge is taken to seal above a threshold (15-20 % in published calibrations; `sgr_threshold_pct`, cite the calibration you use).
- **SSF** (Lindsay et al. 1993): throw over shale-bed thickness per shale bed; smear is typically continuous below about 4-7.
- **Column height** (Bretan et al. 2003): the across-fault pressure difference from the lowest SGR in a sand-sand window, converted to a column with the fluid densities you give (water and hydrocarbon or CO2, from wells_petrophysics or stated). Burial depth selects the calibration constant.

## Pitfalls

- SGR uses the footwall column from one well; lateral facies changes along the fault are not captured: say so in `limitations`.
- A threshold or calibration borrowed from another basin is an assumption; name its source.
- CO2 densities are much lower than oil at shallow depths: column heights change accordingly; cite the density's source.

## Reporting

- `[measurement, derived]`: "F1 at 1200-1500 m, throw 30 m: SGR 18-91 % (median 40); sand-sand windows 22 % of the interval; lowest SGR in sand-sand 18 % (below the 20 % threshold); supportable column 0 m by Bretan et al. (2003)", `prov:` id, product `fault_seal`, depends on the vshale_log and the throw's source.
- Seal profile with `sg_render(kind=seal)`.
