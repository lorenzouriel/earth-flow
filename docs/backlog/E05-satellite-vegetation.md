# E05 — Satellite Vegetation (NDVI)

**Priority:** Must · **Estimate:** 1d · **Day:** D3 · **Owner:** Dev A
**References:** PRD §7 (Satellite), §25, §31; Architecture §1, §5 (raster), §26; README risk "cloud cover"

## Goal

Cloud-masked Sentinel-2 NDVI for each field over the last 12 months, stored as `core.vegetation_observation`, with a quality score that reflects cloud cover and observation age.

## Source

Sentinel-2 L2A from the public **Earth Search** STAC API (Element84), collection `sentinel-2-l2a`. Bands: `red` (B04), `nir` (B08), `scl` (scene classification).

Valid SCL classes: 4 (vegetation) and 5 (bare soil). All other classes are masked: cloud, cloud shadow, cirrus, saturated, no data, water, snow.

---

### S5.1 — STAC search per field  `Must` `S`
As a data engineer, I want to find all Sentinel-2 scenes covering a field in a time window, so that NDVI can be computed per scene.

**Acceptance criteria**
- `search_scenes(field_geom, start, end, max_cloud=60)` returns STAC items sorted by date.
- Item IDs, datetime and `eo:cloud_cover` are recorded in `raw.sentinel_catalog`.

**Subtasks**
- [ ] `earthflow_geo.stac` helpers (pystac-client)
- [ ] `ingestion/sentinel/search.py`
- [ ] `raw.sentinel_catalog` table

### S5.2 — Masked NDVI per field per scene  `Must` `L`
As the feature pipeline, I want one NDVI statistic per field per scene, so that vegetation trends can be tracked.

**Acceptance criteria**
- For each scene: read only the field's bounding box (windowed COG reads), apply the SCL mask, compute NDVI = (NIR − Red) / (NIR + Red).
- Output per field × scene: `ndvi_mean`, `ndvi_p10`, `ndvi_p90`, `valid_pixel_fraction`.
- Scenes with `valid_pixel_fraction < 0.5` are kept but flagged low quality; scenes below 0.2 are dropped.
- NDVI also aggregated to the field's res-9 H3 cells for the map layer (Should).

**Subtasks**
- [ ] `earthflow_geo.raster`: windowed read with stackstac or odc-stac + rioxarray clip to polygon
- [ ] Apply the L2A scale and offset (processing baseline ≥ 04.00 has a −1000 offset)
- [ ] SCL mask
- [ ] Zonal stats
- [ ] Unit test on a tiny synthetic raster

### S5.3 — Normalize to core.vegetation_observation  `Must` `M`
As the platform, I want NDVI in the canonical observation model with quality, so that it joins with weather on the same keys.

**Acceptance criteria**
- Rows: `cell_id` (field's representative res-9 cell) + `field_id`, `observed_at` (scene time), `variable = ndvi`, `value`, `source = sentinel-2-l2a`, `source_version` (processing baseline), `quality_score`.
- `quality_score` derived from `valid_pixel_fraction` and scene cloud cover (formula in `earthflow_core.quality`).
- 12 months of history computed for the 3 fields.

**Subtasks**
- [ ] `ingestion/sentinel/normalize.py`
- [ ] Quality formula + test
- [ ] Backfill run; record runtime

### S5.4 — Field NDVI baseline  `Should` `S`
As the risk model, I want a per-field NDVI baseline, so that "below normal" means something for that field (PRD §28: "slightly below its normal level").

**Acceptance criteria**
- `features.ndvi_baseline` per field: rolling median over the previous 12 months and same-season median (±45 days of the same day-of-year) where data allows.
- Documented limitation: one year is a short baseline.

**Subtasks**
- [ ] DuckDB query
- [ ] Note in `docs/models/vegetation.md`
