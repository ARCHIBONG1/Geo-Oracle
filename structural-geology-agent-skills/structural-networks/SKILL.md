---
name: structural-networks
description: Fault-network topology (I, Y, X nodes), connectivity, abutting relations and relative ages (V4), fault hierarchy, and the fault_network product. Load before sg_fault_network.
---

# Fault networks

## `sg_fault_network`

- Traces from a `fault_set`; a tip within `tolerance_m` of another trace (not of its tips) is an abutment (Y node); crossing traces give X nodes; free tips are I nodes (Sanderson & Nixon 2015).
- Read: node counts, branches, `connections_per_line` (below about 1 the network is poorly connected: isolated faults compartmentalise less and connect pressure less), `connected_groups`, `isolated_lines`.
- **Relative age** (`relative_age`): the abutting fault formed later than, or at the same time as, the one it abuts; mutual abutments mean a pick artefact or coeval faults and fail V4 until explained. Crossings are undated without offset relations.
- **Hierarchy**: by length, throw (give `throws` from the throw profiles) and how many faults abut each one.

## Pitfalls

- A trace that stops short of another because the picks end is an I node to the tool but may be a Y node in reality: say when the picks limit the topology and ask `seismic_interpretation` for the missing sticks.
- Tolerance matters: use about two cells of the survey; report it.
- Topology from one horizon's traces describes that level; faults can link at depth and not at the surface.

## Reporting

- `[measurement, derived]`: "Network at H1: 4 faults, nodes I 7, Y 1, X 1, 0.5 connections per line; F2 abuts F1 (F2 younger or coeval; V4 pass)", `prov:` id, product `fault_network`.
- Compare the abutment order with the regional tectonic phases (regional_framework) for V4.
