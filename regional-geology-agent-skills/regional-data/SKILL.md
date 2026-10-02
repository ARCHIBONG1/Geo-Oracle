---
name: regional-data
description: Areas of interest and regional extents, the geodata snapshot catalogue, supplied GIS files, grid sampling and point queries, resolution and distance-to-data honesty. Load before any data tool.
---

# Regional data

## The area of interest

- `reg_aoi(aoi, name, crs, buffer_km)`: WKT, GeoJSON, a bbox `minx,miny,maxx,maxy`, or points `x y; x y`, in the CRS stated (EPSG:4326 default; projected well coordinates with their EPSG code).
- **The regional extent** is the AOI buffered by `buffer_km` (default 100 km). Choose it by the question: 100 km for stress and heat flow, 300 km for seismicity, the basin outline when one is supplied. State it in `assumptions`.
- Areas and distances are geodesic; every tool reports the distance from the AOI to the data it used.

## The catalogue

- `reg_dataset_catalogue` lists the snapshots in the store (id, name, version, retrieval date, licence, resolution, citation) and the known datasets not yet fetched (`fetchable`).
- **Fetch on first use**: `reg_fetch_dataset('wsm2016')` and so on downloads the pinned release into the store and freezes it with version, licence, checksum and date; later calls reuse that copy. Large grids run as background jobs: collect them with `get_regional_job_result`.
- **Earthquakes**: `reg_fetch_dataset('usgs_comcat', aoi=<record>, start, min_magnitude)` snapshots the catalogue for the regional extent's bounding box; the id (`usgs_comcat_<hash>`) names the query, and the result tells you the id to use as `source`. `refresh=true` takes a new snapshot when the task asks for current data; the old one stays.
- **Custom datasets** (a national survey's download, a published compilation): `reg_fetch_dataset(<new id>, custom={url, name, version, licence, resolution, kind, columns...})`, with every item taken from the dataset's own documentation, never guessed. A download that fails is reported in `limitations`, not worked around.
- **A dataset that is neither catalogued nor fetchable cannot be used.**
- Datasets are used by id as `source` (e.g. `wsm2016`, `ghfdb2024`, `gem_faults`, `etopo2022`, `globsed`, `usgs_comcat_<region>`). Column mappings (azimuth, quality, heat flow, magnitude) come from the catalogue; override them only when a supplied file uses other names.
- Cite every value as `prov:<id>; <dataset name> <version>` with `source_type` `literature` (a published compilation).

## Supplied files

- `reg_register_layer(path, kind, description, crs?, ...)`: vector (GeoJSON, GeoPackage, shapefile, KML), grid (netCDF, GeoTIFF) or points (CSV with lon/lat). Read in place, converted to EPSG:4326 on use. A file without a CRS needs `crs`; never guess a projected system.
- Describe what the file is and its edition in `description`; it travels into every product's dataset list.
- Documents (reports, papers) are not yours: ask literature_review.

## Grids and points

- `reg_grid_sample(source, aoi, mode)`: `stats` in the extent (with the number of cells inside: few cells mean the grid barely resolves the extent), `points` (bilinear), `profile` (a table, drawable with `reg_render`). The cell size in km is reported: a 5 arc-minute grid is about 9 km, a 1 degree grid about 110 km.
- `reg_points_near(source, aoi, radius_km, value_field, filters)`: counts, nearest distance, distance percentiles, value statistics; the subset is saved as a table for maps.

## Resolution and distance

- State in `limitations` the cell size of every grid and the distance to the nearest point of every point dataset. A regional value with no data within 50 km of the AOI does not meet T3 at study-area scale.
- Kilometre-scale grids never support prospect-scale statements.
- A tool warning that a dataset has no points in the radius means unconstrained, not zero.
