# E03 — Field & Crop Registry

**Priority:** Must · **Estimate:** 0.5d · **Day:** D2 · **Owner:** Dev B
**References:** PRD §14, §24–25, §38; README decisions D7, D9

## Goal

Three predefined **arabica coffee** fields in São Paulo state, with their crop parameters and soil assumptions, loaded into both Postgres (`app.field`) and Iceberg (`core.field`, `core.crop`).

## Demo fields

Pick three real coffee areas in different São Paulo coffee regions so the demo can show contrasting conditions. Candidate regions: **Alta Mogiana** (around Franca), **Média Mogiana** (around Espírito Santo do Pinhal), **Centro-Oeste Paulista** (around Garça/Marília).

Polygons are traced by hand over satellite basemap imagery and checked against MapBiomas coffee land cover. Do not invent coordinates.

---

### S3.1 — Trace and store demo field polygons  `Must` `M`
As a demo user, I want three realistic coffee fields on the map, so that the demo feels grounded in São Paulo.

**Acceptance criteria**
- `data/seed/fields.geojson` with 3 polygons; each has `field_id`, `name`, `farm_name` (fictional), `municipality`, `crop_id = coffee_arabica`, `planting_year`, `irrigated: true`, `area_ha` (computed).
- Each polygon is 10–60 ha and visually coffee rows on satellite imagery.
- Area computed in an equal-area CRS, not degrees.

**Subtasks**
- [ ] Trace polygons (geojson.io or QGIS) over imagery
- [ ] Cross-check against MapBiomas coffee class for the same location
- [ ] Compute `area_ha` and H3 cells with `earthflow_geo`
- [ ] Note in the file that farm names are fictional (PRD §38: real exact boundaries are private data)

### S3.2 — Coffee crop configuration  `Must` `M`
As the water and risk models, I want coffee parameters in one versioned file with citations, so that every number is traceable (PRD §39).

**Acceptance criteria**
- `packages/earthflow-agriculture/config/coffee.yaml` contains:
  - Crop coefficients by stage (FAO-56 Table 12, coffee)
  - Root depth `Zr` and depletion fraction `p` (FAO-56 Table 22)
  - Phenology calendar for São Paulo arabica: flowering (Sep–Oct), fruit set / expansion (Nov–Feb), maturation (Mar–May), harvest / rest (May–Aug)
  - Heat stress thresholds (`tmax_stress_c`, `tmax_critical_c`), with flowering-stage overrides
  - Version string `coffee_params_v0.1`
- Every value has a `source:` entry; unverified values are marked `verified: false`.

**Subtasks**
- [ ] Collect FAO-56 values (Kc, Zr, p) for coffee
- [ ] Collect heat thresholds from agronomic literature (Embrapa / IAC)
- [ ] `earthflow_agriculture.crop.load_crop("coffee_arabica")` returning a typed model
- [ ] `crop_stage(date)` function + tests

### S3.3 — Soil profile assumption per field  `Must` `S`
As the water balance model, I want a soil water-holding capacity per field, so that available soil water can be estimated without soil ingestion (decision D7).

**Acceptance criteria**
- Each field has `soil_texture`, `field_capacity`, `wilting_point` (m³/m³) and derived `taw_mm = 1000 × (FC − WP) × Zr`.
- Values come from documented texture defaults (FAO-56 Table 19) and are labelled `assumption` in lineage.

**Subtasks**
- [ ] Add soil attributes to `data/seed/fields.geojson` (or `data/seed/soil.yaml`)
- [ ] `earthflow_water.soil.taw_mm()` + test

### S3.4 — Load registry into Postgres and Iceberg  `Must` `S`
As the platform, I want fields and crop loaded into both stores, so that the API and pipelines read the same registry.

**Acceptance criteria**
- `app.field` (PostGIS geometry) has 3 rows.
- `core.field` and `core.crop` exist in Polaris with the same IDs.
- Re-running the loader is idempotent.

**Subtasks**
- [ ] `ingestion/registry/load_fields.py` (later wrapped as a Dagster asset in E08)
- [ ] Upsert into Postgres; overwrite in Iceberg
