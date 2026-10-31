# E07 — Products, Risk, Provenance & Quality

**Priority:** Must · **Estimate:** 1d · **Day:** D4 · **Owner:** Dev B
**References:** PRD §8.4, §16, §25–27, §29–31, §39; Architecture §28; README decisions D8, D12

## Goal

The three MVP products that applications consume (PRD §25). Every row carries model version, quality, confidence and lineage.

| Product | Grain | Consumed by |
|---|---|---|
| `products.field_state` | field × date | Map card, Copilot |
| `products.irrigation_recommendation` | field × issue date | Water Optimizer, Copilot |
| `products.field_risk` | field × date × risk type | Map card, Copilot |

---

### S7.1 — Risk scoring functions  `Must` `L`
As a farmer, I want my field's risks ranked with evidence, so that I know what to worry about this week (PRD §26 B).

**Acceptance criteria**
- Four scores, each 0–100 with a level (`low < 34 ≤ medium < 67 ≤ high`), computed for the next 7 days:
  - **Heat** — forecast Tmax days above the coffee stress/critical thresholds (stage-aware: flowering is more sensitive).
  - **Water** — forecast soil-water depletion vs RAW/TAW from `features.water_balance`.
  - **Rainfall deficit** — 30-day rainfall vs the climate normal (anomaly %).
  - **Vegetation** — current NDVI vs the field baseline, plus the recent trend.
- **Drought** = documented combination of water + rainfall deficit (PRD §27 shows it separately).
- Each score returns an `evidence` list of `{metric, value, unit, reference, kind}`, e.g. `{rainfall_anomaly_30d, -37, %, normal 1991–2020, reanalysis}`. This powers the "Why?" view (PRD §29).
- Thresholds live in `coffee.yaml`; version `field_risk_v0.1`.

**Subtasks**
- [ ] `earthflow_agriculture.risk` module with one function per risk
- [ ] Evidence model
- [ ] Tests per risk with synthetic inputs crossing each threshold
- [ ] `docs/models/field_risk.md`

### S7.2 — Irrigation recommendation  `Must` `M`
As a farmer, I want to know when and how much to irrigate and how much water I save, so that I don't irrigate before rain (PRD §16, §27 step 5).

**Acceptance criteria**
- Output per field: `recommended_mm` (net, next 7 days), `timing` (`now | delay_Nh | not_needed`), `forecast_rain_mm`, `crop_demand_mm` (ETc over 7 days), `soil_water_pct`, `volume_liters = recommended_mm × area_m²`, `baseline_mm` (fixed schedule, decision D8), `avoided_liters = max(0, baseline − recommended) × area_m²`, `confidence`.
- Delay rule: if forecast rain ≥ X mm within 48 h and depletion stays below RAW until then → `delay_48h` (thresholds configurable).
- Calculated only from `features.water_balance` + forecast. **No LLM involvement** (PRD §16 "Important requirement").
- Version `irrigation_v0.1`.

**Subtasks**
- [ ] `earthflow_water.irrigation.recommend()` + tests (rain-coming case, dry case, saturated case)
- [ ] Baseline schedule config
- [ ] Liters conversion test (1 mm × 1 ha = 10,000 L)

### S7.3 — Field state product  `Must` `M`
As an application, I want one row summarizing a field today, so that the map card renders from a single read (PRD §27 step 3).

**Acceptance criteria**
- Schema per E02 S2.3, including:
  - `vegetation_health` (0–100) — NDVI relative to baseline
  - `water_status` (0–100) — `soil_water_pct` scaled
  - `heat_score`, `drought_score`
- Values agree with `field_risk` and `water_balance` for the same date.

**Subtasks**
- [ ] Builder `products/field_state/build.py`
- [ ] Scaling functions documented

### S7.4 — Quality & confidence propagation  `Must` `M`
As a user, I want to see how reliable each number is, so that the app avoids false precision (PRD §4.3, §31).

**Acceptance criteria**
- Product `quality_score` = documented combination of input qualities: NDVI age and validity, missing weather days, forecast horizon, soil assumption penalty.
- `confidence` mapped from the score; factors kept in `quality.factors`.
- An NDVI older than 30 days lowers vegetation confidence and is explained in evidence.

**Subtasks**
- [ ] `earthflow_core.quality.combine()` + tests
- [ ] Apply in all 3 builders

### S7.5 — Lineage on every product row  `Must` `S`
As a scientist, I want each recommendation to trace back to model versions and datasets, so that results are auditable (PRD §30).

**Acceptance criteria**
- `lineage` JSON includes: product model version, upstream feature versions, `coffee_params` version, each source dataset with time range and fetch time, and `generated_at`.
- A helper renders it as the PRD §30 text block (used by API and UI).

**Subtasks**
- [ ] `earthflow_core.provenance.build_lineage()`
- [ ] `render_lineage_text()` + snapshot test
