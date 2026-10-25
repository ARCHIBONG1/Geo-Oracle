---
name: structural-geometry
description: Orientation statistics (Fisher, orientation tensor), mean planes and sets, fold axes by the pi method, and horizon surface attributes; conventions and pitfalls. Load before sg_orientation_stats, sg_fold_axis or sg_surface_attributes.
---

# Geometry and orientation

## Conventions

Planes as dip and dip azimuth (direction of dip, from north). Lines as plunge and trend. Poles are downward normals. Strike = dip azimuth - 90.

## `sg_orientation_stats`

- Fisher: mean vector, resultant length, concentration k (above about 50 is tight; below 10 is diffuse), alpha95 (the cone of confidence on the mean).
- Orientation tensor: Woodcock's k and C. k > 1 is a cluster (one set), and k < 1 a girdle (folded beds or a conjugate pair). C gives the strength.
- `cluster_threshold_deg` groups the measurements into sets (greedy angular clustering, deterministic for a fixed input order). Report each set's count, mean plane, k and alpha95. The tool leaves sets with fewer than three members unassigned.
- If there are fewer than five orientations, say that the statistics are weakly constrained.

## `sg_fold_axis`

Bedding poles on a cylindrical fold lie on a great circle. The pole of that circle is the fold axis (pi method). Read `cylindricity` (girdle supports a cylindrical fold), the girdle misfit, and k. A cluster means the beds are not folded about one axis in that domain.

## `sg_surface_attributes`

Dip, dip azimuth and mean curvature (Roberts 2001 eq. 15; positive over antiforms, negative over synforms, z down) of a depth horizon grid. The tool refuses time-domain grids. Curvature is a geometric proxy, not fracture intensity: say so if it is used that way. The steepest dips mark fault scarps or the limbs of folds. Compare them with the fault set before you read them as either.

## Reporting

- `[measurement, derived]`: "Fault set: mean plane 085/88, k 120, alpha95 3 deg, n 6; two sets at 085 and 160 (threshold 25 deg)", with the `prov:` id and the product reference in `depends_on`.
- `sg_orientation_stats` draws the stereonet with the mean plane and, for planes, the rose of strikes itself (`result.figures`).
- `sg_fold_axis` draws no figure. Draw its bedding with `sg_render(kind=stereonet)` on the same table.

<!-- end of skill: structural-geometry -->
