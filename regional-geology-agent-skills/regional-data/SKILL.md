---
name: regional-data
description: Areas of interest and regional extents, the geodata snapshot catalogue, supplied GIS files, grid sampling and point queries, resolution and distance-to-data honesty. Load before any data tool.
---

# Regional data

## The area of interest

- `reg_aoi(aoi, name, crs, buffer_km)`: WKT, GeoJSON, a bbox `minx,miny,maxx,maxy`, or points `x y; x y`, in the CRS stated (EPSG:4326 default; projected well coordinates with their EPSG code).
- **Use the location exactly as given.** If `reg_aoi` refuses a coordinate, report the failure under `missing_data`, and stop the location-dependent steps. Never substitute an approximate point.
  - A guessed location moves every distance-dependent result: a guessed point returns another area's stress indicators and earthquakes.
  - The result's `aoi_centroid_lonlat` is the given point in WGS84. Check it against the task's coordinates.
- **The regional extent** is the AOI buffered by `buffer_km` (default 100 km). Choose it by the question: 100 km for stress and heat flow, 300 km for seismicity, the basin outline when one is supplied. State it in `assumptions`.
- Areas and distances are geodesic. Every tool reports the distance from the AOI to the data it used.
- `reg_aoi` draws a map of the AOI and its extent, and returns it in `result.figures`.

## The catalogue

- `reg_dataset_catalogue` lists the snapshots in the store (id, name, version, retrieval date, licence, resolution, citation) and the known datasets not yet fetched (`fetchable`).
- **Fetch on first use**: `reg_fetch_dataset('wsm2016')` and similar calls download the pinned release into the store. The tool freezes it with version, licence, checksum and date. Later calls reuse that copy. Large grids run as background jobs: collect them with `get_regional_job_result`.
- **Earthquakes**: `reg_fetch_dataset('usgs_comcat', aoi=<record>, start, min_magnitude)` snapshots the catalogue for the regional extent's bounding box.
  - The id (`usgs_comcat_<hash>`) names the query. The result tells you the id to use as `source`.
  - If the task asks for current data, `refresh=true` takes a new snapshot. The old one stays.
- **Custom datasets** (a national survey's download, a published compilation): `reg_fetch_dataset(<new id>, custom={url, name, version, licence, resolution, kind, columns...})`. Take every item from the dataset's own documentation, never guess it. Report a download that fails in `limitations`, and do not work around it.
- **A dataset that is neither catalogued nor fetchable cannot be used.**
- Use datasets by id as `source` (e.g. `wsm2016`, `ghfdb2024`, `gem_faults`, `etopo2022`, `globsed`, `usgs_comcat_<region>`). Column mappings (azimuth, quality, heat flow, magnitude) come from the catalogue. Override them only when a supplied file uses other names.
- Cite every value as `prov:<id>; <dataset name> <version>` with `source_type` `literature` (a published compilation).

## Supplied files

- `reg_register_layer(path, kind, description, crs?, ...)`: vector (GeoJSON, GeoPackage, shapefile, KML), grid (netCDF, GeoTIFF) or points (CSV with lon/lat). The tool reads the file in place and converts it to EPSG:4326 on use. A file without a CRS needs `crs`. Never guess a projected system.
- Describe what the file is and its edition in `description`. The description travels into every product's dataset list.
- Documents (reports, papers) are not yours: ask literature_review.

## Grids and points

- `reg_grid_sample(source, aoi, mode)`:
  - `stats` in the extent, with the number of cells inside. Few cells mean that the grid barely resolves the extent.
  - `points` (bilinear).
  - `profile` (a table). The tool draws it and returns the figure in `result.figures`.

  The tool reports the cell size in km: a 5 arc-minute grid is about 9 km, a 1 degree grid about 110 km.
- `reg_points_near(source, aoi, radius_km, value_field, filters)`: counts, nearest distance, distance percentiles, value statistics. The tool saves the subset as a table for maps.

## Resolution and distance

- State in `limitations` the cell size of every grid and the distance to the nearest point of every point dataset. A regional value with no data within 50 km of the AOI does not meet T3 at study-area scale.
- Kilometre-scale grids never support prospect-scale statements.
- If a tool warns that a dataset has no points in the radius, the value is unconstrained, not zero.

<!-- end of skill: regional-data -->
