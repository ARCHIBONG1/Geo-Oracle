---
name: regional-potential-fields
description: Gravity and magnetic grid transforms, lineaments as observations, depth-to-source estimates and their assumptions. Load before reg_grid_transforms, reg_lineaments or reg_depth_to_basement.
---

# Potential fields

## Transforms: `reg_grid_transforms`

- Upward continuation (suppresses short wavelengths; use it before reading regional trends), vertical derivative and total horizontal gradient (sharpen edges), tilt derivative (edges at zero, amplitude-normalised).
- The grids are planar approximations of lon/lat cells at the central latitude; fine within a few hundred kilometres.
- Draw them with `reg_render(grid=<the transforms file>, anomaly=true)`: anomalies get a diverging, zero-centred colour map.

## Lineaments: `reg_lineaments`

- Ridges of the horizontal gradient above a percentile, as segments with azimuth and length, and length-weighted trend statistics.
- **Observations only.** A lineament becomes a fault in your findings only with corroboration: the fault database, a supplied fault map, seismic interpretation or stress data. Report the trend statistics as `[regional]` measurements and the fault reading as an interpretation with its support.

## Depth to source: `reg_depth_to_basement`

- **Spectral**: the radially averaged power spectrum's slope over a wavenumber band gives the depth of the dominant source ensemble (Spector & Grant). One number for the extent, not a map; the band chosen matters, and it is reported.
- **Euler**: windowed solutions with a structural index you state (0 contact, 1 dyke or sill edge, 2 line source, 3 point). Depths change with the index; report the p10-p90 of accepted solutions and the index used. A best-fit plane is removed first.
- Depths are below the observation surface (sea level for marine grids). For depth to basement in a sedimentary basin, compare with the sediment-thickness grid and any well that reached basement; disagreement is a finding.
