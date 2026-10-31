# E04 — Weather Ingestion

**Priority:** Must · **Estimate:** 1d · **Day:** D2–3 · **Owner:** Dev A
**References:** PRD §7 (Weather), §8.1–8.2, §25; Architecture §1; README decisions D5, D9

## Goal

Daily weather history, a 16-day forecast and 1991–2020 climate normals for each field, stored raw and then normalized into `core.weather_observation`.

## Source

**Open-Meteo** (no API key):
- Forecast API — up to 16 days, daily variables
- Historical / archive API (ERA5 reanalysis) — history and normals

Daily variables: `temperature_2m_max`, `temperature_2m_min`, `temperature_2m_mean`, `precipitation_sum`, `relative_humidity_2m_mean`, `wind_speed_10m_max`, `shortwave_radiation_sum`, `et0_fao_evapotranspiration`.

Query point: the centroid of each field's res-7 H3 parent cell (decision D9), so neighbouring fields share weather data.

---

### S4.1 — Open-Meteo client  `Must` `M`
As a data engineer, I want a small, tested client for the Open-Meteo forecast and archive endpoints, so that ingestion code stays simple.

**Acceptance criteria**
- `fetch_daily(lat, lon, start, end, variables, kind)` where `kind ∈ {forecast, archive}`.
- Retries with backoff on HTTP 429/5xx; timeout set.
- Timezone set to UTC in the request; dates returned as UTC dates.
- Tests use recorded responses (no network in CI).

**Subtasks**
- [ ] `ingestion/open_meteo/client.py` (httpx)
- [ ] Record 2 fixture responses
- [ ] Tests

### S4.2 — Raw weather landing  `Must` `S`
As an auditor, I want the untouched API responses kept, so that results can be reproduced and debugged (PRD §8.1).

**Acceptance criteria**
- Each fetch is stored as JSON in `s3://earthflow-raw/open_meteo/{kind}/{cell_id}/{fetch_date}.json`.
- `raw.open_meteo_daily` Iceberg table records request params, cell, fetch time and object path.

**Subtasks**
- [ ] S3 writer in `earthflow_core.storage` (boto3/obstore to RustFS)
- [ ] `raw.open_meteo_daily` table creation

### S4.3 — Normalize to core.weather_observation  `Must` `M`
As a feature pipeline, I want weather in the canonical observation model, so that features don't care about the source format.

**Acceptance criteria**
- One row per `cell_id × date × variable × kind`.
- `kind = reanalysis` for archive, `forecast` for forecast; forecast rows carry `issued_at`.
- Units follow the E02 vocabulary; `source = open-meteo`, `source_version` = model name from the response.
- Re-running for the same window does not duplicate rows (overwrite by partition).

**Subtasks**
- [ ] `ingestion/open_meteo/normalize.py`
- [ ] Partition overwrite for `(day, kind)`
- [ ] Tests on fixture → expected rows

### S4.4 — Backfill and climate normals  `Must` `M`
As the risk model, I want historical weather and normals, so that rainfall and temperature anomalies can be computed (PRD §26 Farm Copilot).

**Acceptance criteria**
- 120 days of history up to the snapshot date for every weather cell.
- Daily climatology for 1991–2020 (day-of-year mean for precipitation and Tmax, smoothed with a ±15-day window) stored as `core.weather_normal`.

**Subtasks**
- [ ] Backfill script (chunked by year to respect rate limits)
- [ ] Climatology computation in DuckDB
- [ ] `core.weather_normal` table

### S4.5 — INMET station data  `Could` `M`
As a judge, I want to see a second weather source harmonized into the same model, so that "integrates multiple datasets" is demonstrated (PRD §42.1).

**Acceptance criteria**
- Nearest INMET automatic station per field ingested as `kind = observed`, normalized into `core.weather_observation`.
- Field state prefers station observations when available and records which source was used.

**Subtasks**
- [ ] Find nearest stations to the 3 fields
- [ ] Client + normalize
- [ ] Source-priority rule in E06
