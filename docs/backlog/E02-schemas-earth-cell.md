# E02 — Canonical Schemas & Earth Cell

**Priority:** Must · **Estimate:** 1d · **Day:** D1–2 · **Owner:** Dev B
**References:** PRD §3.2, §4.2–4.3, §8.2, §9, §25, §30–31; Architecture §1, §26, §28

## Goal

Define, once, the data contracts every layer uses: observations, fields, products, provenance and quality. These schemas are platform-independent (Pydantic + JSON Schema) and map 1:1 to Iceberg tables.

## Tables defined by this epic

```text
core.field                    core.crop
core.weather_observation      core.vegetation_observation
features.field_daily          features.water_balance
products.field_state          products.irrigation_recommendation
products.field_risk
```

---

### S2.1 — Observation base schema  `Must` `M`
As a data engineer, I want every observation to share the same base fields, so that weather and vegetation data interoperate (PRD §3.2, §8.2).

**Acceptance criteria**
- `Observation` model has: `cell_id`, `observed_at` (UTC), `valid_from`/`valid_to` (for forecasts), `variable`, `value`, `unit`, `source`, `source_version`, `quality_score`, `ingested_at`.
- `kind` distinguishes `observed | forecast | reanalysis | derived` (PRD §29: users must tell observed facts from forecasts).
- Timestamps are timezone-aware UTC; a validator rejects naive datetimes.

**Subtasks**
- [ ] `earthflow_core.schemas.observation`
- [ ] Controlled vocabulary of variables + units (`temperature_2m_max: °C`, `precipitation: mm`, `et0: mm`, `ndvi: 1`, …)
- [ ] Unit tests for validators

### S2.2 — Provenance & quality schema  `Must` `M`
As a user, I want every derived value to say where it came from and how reliable it is, so that results are credible (PRD §4.2, §30, §31).

**Acceptance criteria**
- `Lineage` model: `product`, `model_version`, `generated_at`, `inputs: list[InputRef]` where `InputRef` = `dataset`, `source`, `source_version`, `time_range`, `fetched_at`.
- `Quality` model: `quality_score` (0–1), `confidence` (`low | medium | high`), `factors` (dict, e.g. `observation_age_days`, `cloud_fraction`, `missing_days`).
- Both serialize to JSON for an Iceberg string column and to Postgres `jsonb`.

**Subtasks**
- [ ] `earthflow_core.schemas.provenance`
- [ ] `earthflow_core.schemas.quality` with `confidence_from_score()` mapping
- [ ] Tests for round-trip serialization

### S2.3 — Field, crop and product schemas  `Must` `M`
As a developer, I want typed models for every MVP table, so that pipelines, API and MCP tools share one contract (Architecture §28).

**Acceptance criteria**
- Pydantic models for all 9 tables listed above.
- `products.field_state` contains at least the fields in Architecture §28 that are in MVP scope: `field_id`, `observed_date`, `h3_cell`, `crop_id`, `crop_stage`, `temperature_mean`, `temperature_max`, `rainfall_7d`, `rainfall_30d`, `soil_water_pct`, `ndvi`, `ndvi_delta`, `water_deficit_mm`, `vegetation_health`, `water_status`, `heat_score`, `drought_score`, `quality_score`, `model_version`, `lineage`, `calculated_at`.
- JSON Schema exported to `packages/earthflow-core/schemas/*.json`.

**Subtasks**
- [ ] `earthflow_core.schemas.field`, `.crop`, `.features`, `.products`
- [ ] `make schemas` exporting JSON Schema
- [ ] Short doc `docs/data-model.md` listing each table, grain and key

### S2.4 — Pydantic → Iceberg schema mapping  `Must` `M`
As a data engineer, I want Iceberg tables created from the Pydantic models, so that the lakehouse never drifts from the contract.

**Acceptance criteria**
- `create_or_update_table(model, namespace, partition_spec)` creates the table in Polaris from the model.
- Partitioning: observations by `day(observed_at)`; field tables by `field_id` / `observed_date`.
- An `append(model_rows)` / `overwrite_partition(...)` helper writes via Arrow.

**Subtasks**
- [ ] Type mapping (str, float, int, datetime UTC, date, JSON-as-string, WKB bytes)
- [ ] `earthflow_core.lakehouse.tables` helpers
- [ ] Test against a local in-memory/SQLite pyiceberg catalog so it runs in CI without Polaris

### S2.5 — Earth Cell (H3) helpers  `Must` `M`
As a data engineer, I want one spatial key across datasets, so that weather and satellite data join on fields (PRD §9, Architecture §1).

**Acceptance criteria**
- `latlon_to_cell(lat, lon, res)`, `polygon_to_cells(geom, res)`, `cell_to_parent(cell, res)`, `cell_to_geometry(cell)`.
- Constants: `FIELD_RES = 9`, `WEATHER_RES = 7` (decision D9).
- Field polygon → set of res-9 cells and its res-7 parent(s).

**Subtasks**
- [ ] `earthflow_geo.h3` module (h3-py v4 API)
- [ ] `earthflow_geo.vector`: WKB/GeoJSON conversion, area in hectares (equal-area CRS via pyproj)
- [ ] Tests with a known polygon
