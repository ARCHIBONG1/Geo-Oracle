---
name: structural-networks
description: Fault-network topology (I, Y, X nodes), connectivity, abutting relations and relative ages (V4), fault hierarchy, and the fault_network product. Load before sg_fault_network.
---

# Fault networks

## `sg_fault_network`

- Traces from a `fault_set`. The nodes (Sanderson & Nixon 2015):
  - a tip within `tolerance_m` of another trace (not of its tips) is an abutment (Y node, also when the tip overshoots);
  - every crossing point is an X node (a pair crossing twice gives two);
  - a tip within the tolerance of another tip is a tip contact: one fault picked as two (counted as one line, listed in `tip_contacts`);
  - free tips are I nodes.
- Read:
  - node counts, branches;
  - `connections_per_branch` (0 to 2): the measure that Sanderson & Nixon 2015 prefer, almost independent of topology. Their well-connected network gives 1.7, a network of local clusters 1.1;
  - `connections_per_line`: depends on topology. 3.57 is the percolation value for random lines of fixed length;
  - `connected_groups`, `isolated_lines`.

  Quote these values with the source, not a threshold of your own.
- **Relative age** (`relative_age`): the abutting fault formed later than, or at the same time as, the one it abuts. Mutual abutments mean a pick artefact or coeval faults, and fail V4 until explained. Without offset relations, crossings are undated.
- **Hierarchy**: by length, throw (give `throws` from the throw profiles) and how many faults abut each one.
- **The product carries the geometry**: each fault's trace, depth, dip and the throw you gave travel in `fault_network` (`faults`). This is what stratigraphy's gap-versus-fault check reads. Publish the product with `throws`, so that a cut-out can be matched.

## Pitfalls

- A trace that stops short of another because the picks end is an I node to the tool, but may be a Y node in reality. Say when the picks limit the topology, and ask `seismic_interpretation` for the missing sticks.
- Tolerance matters. The default is two survey cells: twice the median spacing of the traces' vertices (`tolerance_basis` says so). Report it. If the traces are simplified, give `tolerance_m`.
- Topology from one horizon's traces describes that level. Faults can link at depth and not at the surface.

## Reporting

- `[measurement, derived]`: "Network at Top Reservoir: 4 faults, nodes I 7, Y 1, X 1, 0.5 connections per line; F4 abuts F3 (F4 younger or coeval; V4 pass)", `prov:` id, product `fault_network`.
- Compare the abutment order with the regional tectonic phases (regional_framework) for V4.

<!-- end of skill: structural-networks -->
