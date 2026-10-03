---
name: structural-geometry
description: Orientation statistics (Fisher, orientation tensor), mean planes and sets, fold axes by the pi method, and horizon surface attributes; conventions and pitfalls. Load before sg_orientation_stats, sg_fold_axis or sg_surface_attributes.
---

# Geometry and orientation

## Conventions

Planes as dip and dip azimuth (direction of dip, from north). Lines as plunge and trend. Poles are downward normals. Strike = dip azimuth - 90.

## `sg_orientation_stats`

- Fisher: mean vector, resultant length, concentration k (above about 50 is tight; below 10 is diffuse), alpha95 (the cone of confidence on the mean).
- Orientation tensor: Woodcock's k and C. k > 1 is a cluster (one set), k < 1 a girdle (folded beds or a conjugate pair); C gives the strength.
- `cluster_threshold_deg` groups the measurements into sets (greedy angular clustering, deterministic for a fixed input order). Report each set's count, mean plane, k and alpha95; sets with fewer than three members are left unassigned.
- Fewer than five orientations: say the statistics are weakly constrained.

## `sg_fold_axis`

Bedding poles on a cylindrical fold lie on a great circle; its pole is the fold axis (pi method). Read `cylindricity` (girdle supports a cylindrical fold), the girdle misfit, and k. A cluster means the beds are not folded about one axis in that domain.

## `sg_surface_attributes`

Dip, dip azimuth and mean curvature of a depth horizon grid (time-domain grids are refused). Curvature is a geometric proxy, not fracture intensity: say so if it is used that way. The steepest dips mark fault scarps or the limbs of folds; compare with the fault set before reading them as either.

## Reporting

- `[measurement, derived]`: "Fault set: mean plane 085/88, k 120, alpha95 3 deg, n 6; two sets at 085 and 160 (threshold 25 deg)", with the `prov:` id and the product reference in `depends_on`.
- Stereonet with `sg_render(kind=stereonet)` and the mean plane drawn; rose of strikes with `kind=rose`.
