---
name: regional-potential-fields
description: Gravity and magnetic grid transforms, lineaments as observations, depth-to-source estimates and their assumptions. Load before reg_grid_transforms, reg_lineaments or reg_depth_to_basement.
---

# Potential fields

## Transforms: `reg_grid_transforms`

- Upward continuation (suppresses short wavelengths; use it before reading regional trends), vertical derivative and total horizontal gradient (sharpen edges), tilt derivative (edges at zero, amplitude-normalised).
- Before any transform, the tool resamples the lon/lat grid onto a regular metre grid in the AOI's local equal-area projection. The `note` and `grid` fields say how. Euler solutions and lineament centres come back with lon/lat.
- Draw them with `reg_render(grid=<the transforms file>, anomaly=true)`: anomalies get a diverging, zero-centred colour map.

## Lineaments: `reg_lineaments`

- Ridges of the horizontal gradient above a percentile, as segments with azimuth and length, and length-weighted trend statistics.
- **Observations only.** A lineament becomes a fault in your findings only with corroboration: the fault database, a supplied fault map, seismic interpretation or stress data. Report the trend statistics as `[regional]` measurements, and the fault reading as an interpretation with its support.

## Depth to source: `reg_depth_to_basement`

- **The extent is the grid's, not the stress radius.** `extent_km` sets the analysis extent around the AOI (also for `reg_grid_transforms` and `reg_lineaments`). Without it, the tool uses the AOI record's buffer. If the grid does not cover the extent, the refusal names the largest extent it does cover: rerun with `extent_km` at or below that. A supplied survey grid is usually smaller than the 100-300 km used for stress or seismicity.

- **Spectral**: the slope of the radially averaged power spectrum over a wavenumber band gives the depth of the dominant source ensemble (Spector & Grant). It is one number for the extent, not a map. The band chosen matters. The result gives it: state it with the depth.
- **Euler**: windowed solutions with a structural index set by the source you expect.
  - Give `source_type` (sphere, pipe, dyke, step, contact). The tool reads the field (gravity or magnetic) from the grid's units, or give `field`.
  - Magnetic: sphere 3, pipe 2, dyke 1, step (sill edge, fault of limited throw) 1, contact 0.
  - Gravity: sphere (point mass) 2, pipe (line of mass) 1, step 1. A gravity dyke or contact has no published index in hand, and the tool refuses it (Reid et al. 1990; Blakely 1995 Table 10.1).
  - An explicit `structural_index` overrides `source_type`; cite its source. With neither, the tool assumes 1 and warns.
  - Depths change with the index. Report the p10-p90 of accepted solutions, the index and its basis (`structural_index_basis`).
  - Nothing is removed first: the regional is Reid's constant background in each window.
  - `plane_variance_share` says how much of the grid a plane explains. Above half, the tool warns, because that is either a linear regional or a contact crossing the grid. For a regional, rerun with `regional="plane"` (a Geo Oracle option, stated in the result), or remove a regional first. For a contact or fault crossing the grid, keep the default, since removing the plane misplaces it.
  - The tool accepts a solution, as Reid et al. (1990) do, when the standard deviation of its depth is below 15 % of the depth (`acceptance` says so). At index 0, the tool solves Reid's eq. 2 with an offset.
  - Along a straight contact or dyke, the position along strike is undetermined: solutions sit at the window centres along the feature.
- Depths are below the observation surface (sea level for marine grids). For depth to basement in a sedimentary basin, compare with the sediment-thickness grid and any well that reached basement. A disagreement is a finding.

<!-- end of skill: regional-potential-fields -->
