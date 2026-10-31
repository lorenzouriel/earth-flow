# E06 — Features: field_daily & water_balance

**Priority:** Must · **Estimate:** 1.5d · **Day:** D3–4 · **Owner:** Dev B
**References:** PRD §3.4, §8.3, §14, §16, §25, §39; README decisions D6, D7, D8

## Goal

Deterministic, versioned, unit-tested models that turn observations into:
- `features.field_daily` — one row per field per day: weather, NDVI and crop stage aligned.
- `features.water_balance` — FAO-56 daily soil-water balance per field: ETc, effective rain, depletion, deficit.

All logic lives in `packages/earthflow-water` and `packages/earthflow-agriculture`. Pipelines only call these packages (Architecture §26).

## Model: `water_balance_v0.1` (FAO-56 single crop coefficient, daily root-zone depletion)

```text
ETc_d    = Kc(stage) × ET0_d
Pe_d     = effective rainfall (fixed-fraction or USDA-SCS method, documented)
Dr_d     = clamp(Dr_{d-1} − Pe_d − I_d + ETc_d, 0, TAW)
RAW      = p × TAW
stress   = Dr_d > RAW
deficit  = Dr_d (mm needed to refill to field capacity)
```

Initial condition: start 120 days before the snapshot at field capacity (`Dr = 0`), long enough for the warm-up to wash out. This is documented as an assumption.

---

### S6.1 — Reference evapotranspiration (ET0)  `Should` `M`
As the water model, I want ET0 computed by Earth Flow itself, so that the core calculation is owned, deterministic and testable (decision D6).

**Acceptance criteria**
- `et0_penman_monteith(tmin, tmax, rh_mean, wind_2m, rs, lat, elevation, doy)` per FAO-56 Eq. 6.
- Wind converted from 10 m to 2 m (FAO-56 Eq. 47).
- Passes the FAO-56 worked example (Example 18) within ±0.1 mm/day.
- Cross-check against Open-Meteo `et0_fao_evapotranspiration` over the backfill; mean absolute difference logged.
- Fallback: if inputs are missing, use Open-Meteo ET0 and record that in lineage.

**Subtasks**
- [ ] `earthflow_water.et0` module
- [ ] Test with FAO-56 Example 18 values
- [ ] Cross-check notebook/script

### S6.2 — Crop coefficient and crop stage per day  `Must` `S`
As the water model, I want Kc for each field-day, so that crop water demand reflects coffee phenology.

**Acceptance criteria**
- `kc_for(date, crop)` uses the `coffee.yaml` stage calendar (E03 S3.2).
- Stage and Kc version are recorded in output rows.

**Subtasks**
- [ ] `earthflow_agriculture.crop.kc_for()`
- [ ] Tests for stage boundaries

### S6.3 — Daily water balance model  `Must` `L`
As a farmer, I want to know how much water my field is missing, so that I can decide whether to irrigate (PRD §16).

**Acceptance criteria**
- Given a field, a daily series of ET0 and rainfall, Kc and TAW, returns daily `etc_mm`, `pe_mm`, `depletion_mm`, `soil_water_pct = 1 − Dr/TAW`, `stress_flag`.
- Supports recorded irrigation events (empty list in the MVP) and forecast days (continues the series into the forecast).
- Pure function; no I/O.
- Model version string `water_balance_v0.1`.
- Tests: no rain → depletion grows by ETc each day until TAW; heavy rain → depletion goes to 0, excess counted as drainage; known hand-calculated 10-day case.

**Subtasks**
- [ ] `earthflow_water.balance.run_water_balance()`
- [ ] Effective-rainfall function (documented choice)
- [ ] Hand-calculated test fixture
- [ ] `docs/models/water_balance.md`: assumptions, equations, datasets, limitations (PRD §39)

### S6.4 — Build features.field_daily  `Must` `M`
As the products layer, I want one aligned row per field per day, so that products don't redo joins.

**Acceptance criteria**
- Columns: `field_id`, `date`, `crop_stage`, `tmin`, `tmax`, `tmean`, `precip_mm`, `rh_mean`, `et0_mm`, `ndvi` (latest valid scene ≤ date), `ndvi_age_days`, `ndvi_baseline`, `weather_kind` (`reanalysis | observed | forecast`), `quality_score`, `lineage`.
- Covers history (120 days) plus forecast (16 days).
- Builds from Iceberg via DuckDB or pyiceberg → Arrow; writes back to `features.field_daily`.

**Subtasks**
- [ ] `products/…/build_field_daily.py` (or `earthflow_agriculture.features`)
- [ ] As-of join for NDVI (last valid observation)
- [ ] Source-priority rule if INMET is present (E04 S4.5)

### S6.5 — Build features.water_balance  `Must` `S`
As the irrigation product, I want the water balance materialized per field per day, so that the API reads results instead of recomputing them.

**Acceptance criteria**
- `features.water_balance` rows for history + forecast with `model_version` and lineage pointing at the `field_daily` inputs.
- Deterministic: two runs on the same input give identical output.

**Subtasks**
- [ ] Builder calling `run_water_balance()`
- [ ] Determinism test
