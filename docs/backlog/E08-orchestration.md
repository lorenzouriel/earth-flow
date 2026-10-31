# E08 — Orchestration (Dagster) & Serving Sync

**Priority:** Must · **Estimate:** 0.5d · **Day:** D4 · **Owner:** Dev A
**References:** Architecture §20, §23; README decision D4

## Goal

The whole pipeline is one Dagster asset graph (Architecture §20). The final products are synced into Postgres serving tables for the API.

```text
field_registry ─┬─► weather_raw ─► weather_observation ─┐
                │   weather_normals ────────────────────┤
                └─► sentinel_catalog ─► vegetation_obs ─┼─► field_daily ─► water_balance ─┬─► irrigation_recommendation ─┐
                                        ndvi_baseline ──┘                                 ├─► field_risk ─────────────────┼─► serving_sync
                                                                                          └─► field_state ────────────────┘
```

---

### S8.1 — Asset definitions  `Must` `M`
As an operator, I want every table to be a Dagster asset with explicit dependencies, so that the lineage graph is visible and re-runnable.

**Acceptance criteria**
- One asset per table in the graph above, each calling package/ingestion code (no domain logic inside asset functions).
- Daily partitions for observation and feature assets.
- A full materialization from the UI succeeds end-to-end.

**Subtasks**
- [ ] `platforms/local/dagster/assets/*.py`
- [ ] Resources: Polaris catalog, S3 (RustFS), Postgres, settings
- [ ] Asset metadata: row counts, model version, quality summary

### S8.2 — Serving sync to Postgres  `Must` `S`
As the API, I want the latest products in Postgres, so that reads are fast and simple (Architecture §23).

**Acceptance criteria**
- `serving.field_state`, `serving.irrigation_recommendation`, `serving.field_risk`, `serving.field_daily` (for charts) are upserted from Iceberg after each run.
- `lineage` and `quality` stored as `jsonb`.

**Subtasks**
- [ ] Sync asset (Arrow → Postgres via `COPY` or `executemany`)
- [ ] Migrations for serving tables

### S8.3 — Asset checks  `Should` `S`
As an operator, I want basic data checks, so that bad data fails loudly before the demo.

**Acceptance criteria**
- Checks:
  - No duplicate keys
  - NDVI in [-1, 1]
  - Precipitation ≥ 0
  - Every product row has `model_version` and non-empty lineage
  - Forecast covers ≥ 7 days

**Subtasks**
- [ ] Dagster `@asset_check`s for the listed rules

### S8.4 — Daily schedule  `Could` `S`
As an operator, I want a daily refresh, so that the platform stays fresh after the hackathon.

**Acceptance criteria**
- Schedule at 06:00 America/Sao_Paulo; disabled in demo/snapshot mode.

**Subtasks**
- [ ] Dagster schedule gated by a `DEMO_SNAPSHOT_DATE` setting
