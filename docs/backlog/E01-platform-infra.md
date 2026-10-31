# E01 — Local Platform Infrastructure

**Priority:** Must · **Estimate:** 1.5d · **Day:** D1–2 · **Owner:** Dev A
**References:** Architecture §16–20, §22–23; README decision D2, D3, D4

## Goal

One `docker compose up` brings up the Local/OSS reference stack used by the MVP: **RustFS** (S3 storage), **Apache Polaris** (Iceberg REST catalog), **Trino** (SQL), **Postgres/PostGIS** (operational and serving data) and **Dagster** (orchestration).

**Polaris is required (decision D2).** The D1 exit gate is S1.3: an Iceberg table written and read back through Polaris.

## Out of scope

Kafka, Flink, Spark, Sedona, Oxigraph, Qdrant, MLflow (Architecture §21: not for the first hackathon build).

---

### S1.1 — Object storage with RustFS  `Must` `S`
As the platform, I want S3-compatible object storage, so that raw files and Iceberg data files have a home.

**Acceptance criteria**
- Given `docker compose up`, RustFS is reachable on its S3 endpoint and console.
- Buckets `earthflow-raw` and `earthflow-lakehouse` exist after startup.

**Subtasks**
- [ ] `platforms/local/compose.yaml` with RustFS service + named volume
- [ ] Init container/script creating buckets and an access key for Polaris
- [ ] Record pinned image tag in `platforms/local/VERSIONS.md`

### S1.2 — Apache Polaris catalog  `Must` `L`
As the platform, I want an Iceberg REST catalog, so that every engine (pyiceberg, Trino) sees the same tables.

**Acceptance criteria**
- Polaris runs in Compose with persistent metadata storage (not in-memory).
- A catalog `earthflow` exists with its storage location in `s3://earthflow-lakehouse/`, pointing at the RustFS endpoint (path-style access).
- Namespaces `raw`, `core`, `features`, `products` exist.
- A principal with credentials for pyiceberg and Trino is created by a bootstrap script, never by hand.

**Subtasks**
- [ ] Start from Polaris' official local getting-started guide (the RustFS variant); pin the release version
- [ ] Configure Polaris metadata persistence (relational backend on the shared Postgres, separate database `polaris`)
- [ ] Bootstrap script `platforms/local/polaris/bootstrap.sh`: realm, root credentials, catalog, principal, principal role, catalog role, grants
- [ ] Configure S3 endpoint, region and path-style access for RustFS
- [ ] Create the 4 namespaces
- [ ] Write down every gotcha hit during setup in `platforms/local/polaris/README.md`

### S1.3 — Iceberg round-trip test (D1 exit gate)  `Must` `S`
As a developer, I want an automated check that writing and reading Iceberg through Polaris works, so that storage problems surface immediately.

**Acceptance criteria**
- Given the stack is up, `make smoke` creates `raw._smoke`, appends 3 rows with pyiceberg, reads them back, and drops the table.
- The same table is visible from Trino while it exists.

**Subtasks**
- [ ] `earthflow_core.lakehouse` module: `get_catalog()` returning a pyiceberg REST catalog from settings
- [ ] `platforms/local/smoke_test.py`
- [ ] `make smoke` target

### S1.4 — Trino query engine  `Should` `M`
As a data scientist, I want SQL over the Iceberg tables, so that the demo can show the lakehouse is queryable by any engine.

**Acceptance criteria**
- Trino's `iceberg` catalog uses the Polaris REST catalog and the RustFS S3 endpoint.
- `SELECT * FROM iceberg.core.field` works once E03 has run.

**Subtasks**
- [ ] `platforms/local/trino/catalog/iceberg.properties` (`iceberg.catalog.type=rest`, Polaris URI, OAuth2 credential, S3 settings)
- [ ] Add Trino to Compose
- [ ] Saved demo query `docs/demo/queries.sql` (used in E12)

### S1.5 — Postgres + PostGIS  `Must` `S`
As the API, I want an operational database with spatial support, so that field polygons and serving tables are fast to query.

**Acceptance criteria**
- Postgres with PostGIS enabled; databases `earthflow` (app/serving) and `polaris` (catalog metadata).
- Schema migrations run from code (Alembic or plain SQL files applied on start).

**Subtasks**
- [ ] PostGIS image in Compose + init SQL for both databases
- [ ] Migration tooling in `apps/api`
- [ ] Initial schema: `app.field` (geometry), `serving.*` placeholder

### S1.6 — Dagster service  `Must` `S`
As the platform, I want Dagster running in Compose, so that assets can be materialized from its UI (filled in E08).

**Acceptance criteria**
- Dagster webserver + daemon up; UI shows the `earthflow` code location.

**Subtasks**
- [ ] `platforms/local/dagster/` with `definitions.py` stub and `dagster.yaml` (Postgres run storage)
- [ ] Compose services for webserver and daemon

### S1.7 — One-command startup  `Must` `S`
As a developer, I want `make up` to start everything in the right order, so that setup works the same on both machines.

**Acceptance criteria**
- `make up` → all services healthy, buckets and catalog bootstrapped, smoke test passes.
- `make down` stops everything; `make reset` also wipes volumes.

**Subtasks**
- [ ] Healthchecks and `depends_on: condition: service_healthy`
- [ ] `make up / down / reset / smoke`
- [ ] Document ports and UIs in `platforms/local/README.md`
