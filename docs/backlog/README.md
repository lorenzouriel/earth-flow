# Earth Flow — Hackathon MVP Backlog

**Source documents:** [PRD](../PRD.md) · [Architecture](../ARCHITECTURE.md)
**Target:** Reboot the Earth São Paulo 2026
**Team / time:** 1–2 developers, ~7 days
**Deployment profile:** Local / OSS (Architecture §16–25)

---

## 1. MVP definition

> Prove the platform thesis: several environmental datasets become reusable data products, and **two separate applications — Water Optimizer and Farm Copilot — consume the same products** to reach specific, quantified decisions for a coffee field in São Paulo.

The judges must understand this in about one minute (PRD §42).

### Demo flow (PRD §27)

1. Open a map of São Paulo.
2. Select one of the predefined coffee fields.
3. See the field state: vegetation health, water status, heat risk, drought risk.
4. Ask *"Preciso irrigar esta semana?"* → water model runs → recommendation in mm and liters avoided.
5. Ask *"Qual o maior risco para essa plantação?"* → Copilot ranks risks with evidence, confidence and sources.
6. Open "Why?" and the provenance view → every number traces to a model version and dataset.

---

## 2. Scope

| In scope | Out of scope (later phases) |
|---|---|
| São Paulo state, **3 predefined coffee fields** | Drawing fields in the app, auth, multi-tenancy |
| **Coffee only** (arabica) | Other crops, crop calendars |
| Open-Meteo: history, 16-day forecast, 1991–2020 normals | INMET (optional stretch), hydrology |
| Sentinel-2 L2A NDVI via public STAC, precomputed per field | Live raster processing, COG/Zarr pipelines, Landsat |
| Fixed soil profile per field (documented assumption) | Soil ingestion (SoilGrids etc.) |
| FAO-56 water balance `water_balance_v0.1` | ML models, disease risk, yield |
| Heat, water, rainfall, vegetation risk → `field_risk` | Full EarthEvent model, notifications |
| Provenance + quality score + confidence on every product | Earth Graph, ontology, RAG knowledge base |
| FastAPI `/v1/fields/*` + MCP server (7 tools) | SDK, public API keys |
| Claude-powered Farm Copilot (PT-BR), "Why?" view | Autonomous actions |
| Next.js + MapLibre web app | deck.gl, mobile |
| RustFS + **Apache Polaris** + Iceberg + Trino + Postgres/PostGIS + Dagster | Kafka, Flink, Spark, Sedona, Oxigraph, Qdrant, MLflow |

---

## 3. Key decisions

| # | Decision | Reason |
|---|---|---|
| D1 | Use the repo layout from Architecture §29, not PRD §36 | Keeps domain logic in portable `packages/`; platform code in `platforms/local/` |
| D2 | **Polaris is required** — no catalog fallback | User decision. Day 1 is dedicated to a working Polaris round-trip before anything else touches storage |
| D3 | Compute with Python + DuckDB, write Iceberg via **pyiceberg** to Polaris | Data is small (3 fields); Spark adds setup cost with no benefit (Architecture §19) |
| D4 | Iceberg = analytical record (queried via Trino); `products.*` synced to Postgres for the API | Architecture §7/§23: lakehouse is not the OLTP database; keeps API latency low |
| D5 | Open-Meteo for weather (forecast + ERA5 archive) | INMET has observations only; demo needs forecast rainfall |
| D6 | ET0 computed in our own package (FAO-56 Penman-Monteith), cross-checked against Open-Meteo's `et0_fao_evapotranspiration` | Deterministic, testable, owned by Earth Flow (PRD §3.4) |
| D7 | Fixed soil profile per field: total available water (TAW) from documented texture defaults, coffee root depth from FAO-56 Table 22 | Water balance needs soil water; soil ingestion is Phase 1 |
| D8 | Water-savings baseline = configurable fixed schedule (default: 20 mm every 7 days) | PRD §16 requires quantified savings; a baseline must be explicit and visible |
| D9 | Earth Cell = H3, resolution 9 for field cells, resolution 7 for weather | Architecture §1 standard; weather grids are coarse |
| D10 | Agent: Claude via Anthropic API, PydanticAI as the agent framework, tools via `earthflow-mcp` | Native MCP support; agent only explains tool outputs (PRD §12, §33) |
| D11 | Demo runs on a **frozen snapshot date** with all data preloaded | Removes live-API and cloud-cover risk during judging |
| D12 | `get_active_events` from PRD §33 becomes `get_field_risks` in the MVP | No event model in scope; risks cover the same need |

---

## 4. Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Polaris setup (bootstrap, credentials, S3 endpoint for RustFS) takes longer than planned | Blocks lakehouse writes | Follow Polaris' own local guide (which uses RustFS); pin versions; S1.3 round-trip test is the D1 exit gate. Domain packages (E2, E6) develop against DuckDB in parallel, so they don't block on infra |
| Cloud cover over São Paulo in the rainy season | No recent NDVI | Use 60–90 day lookback, SCL masking, report `observation_age` and lower quality |
| Coffee thresholds/coefficients wrong | Credibility | Every parameter in one config file with a citation; flag unverified values |
| LLM invents numbers | Violates PRD §12/§33 | System prompt + tool-only facts + test set of questions that check numbers match tool output |
| Live APIs down during demo | Demo fails | Frozen snapshot (D11) + recorded backup video |
| One-developer scenario | ~7 dev-days for ~13 of work | Priority tags below; cut list in §7 |

---

## 5. Epics

| Epic | Title | Priority | Est. | Day | Owner (2 devs) |
|---|---|---|---|---|---|
| [E00](E00-foundation.md) | Repo foundation | Must | 0.5d | D1 | A |
| [E01](E01-platform-infra.md) | Local platform infra (RustFS, Polaris, Trino, Postgres, Dagster) | Must | 1.5d | D1–2 | A |
| [E02](E02-schemas-earth-cell.md) | Canonical schemas & Earth Cell | Must | 1d | D1–2 | B |
| [E03](E03-field-crop-registry.md) | Field & crop registry | Must | 0.5d | D2 | B |
| [E04](E04-weather-ingestion.md) | Weather ingestion | Must | 1d | D2–3 | A |
| [E05](E05-satellite-vegetation.md) | Satellite vegetation (NDVI) | Must | 1d | D3 | A |
| [E06](E06-features-water-balance.md) | Features: field_daily & water_balance | Must | 1.5d | D3–4 | B |
| [E07](E07-products-risk-provenance.md) | Products, risk, provenance & quality | Must | 1d | D4 | B |
| [E08](E08-orchestration.md) | Dagster asset graph & serving sync | Must | 0.5d | D4 | A |
| [E09](E09-api-mcp.md) | FastAPI & MCP server | Must | 1d | D4–5 | B |
| [E10](E10-farm-copilot-agent.md) | Farm Copilot agent | Must | 1d | D5 | B |
| [E11](E11-web-app.md) | Web app (map, Water Optimizer, Copilot) | Must | 1.5d | D5–6 | A |
| [E12](E12-demo-hardening.md) | Demo hardening & narrative | Must | 1d | D6–7 | A+B |

**Total ≈ 13 dev-days** against 14 available with two developers.

### Estimate scale

| Size | Effort |
|---|---|
| S | ≤ 2 hours |
| M | ~ half day |
| L | ~ 1 day |

### Priority tags on stories

- **Must** — demo breaks without it.
- **Should** — visibly improves the demo or credibility.
- **Could** — only if ahead of schedule.

---

## 6. Dependency graph & critical path

```text
E00 ──┬─► E01 (infra) ───────────────┬─► E08 (orchestration) ─► E12
      │                              │
      └─► E02 (schemas) ─► E03 ─┬─► E04 (weather) ─┐
                                └─► E05 (NDVI) ────┼─► E06 (features) ─► E07 (products)
                                                   │                        │
                                                   │                        ▼
                                                   │                  E09 (API + MCP)
                                                   │                   │          │
                                                   │                   ▼          ▼
                                                   │            E10 (agent)   E11 (web)
                                                   │                   └────┬─────┘
                                                   └───────────────────────►E12 (demo)
```

**Critical path:** E02 → E06 → E07 → E09 → E10 / E11 → E12.

E06 and E07 are pure Python and run on DuckDB/Parquet fixtures, so they progress while E01 is still being finished.

---

## 7. Day plan (2 developers)

| Day | Dev A — platform / data / web | Dev B — domain / API / agent | Exit gate |
|---|---|---|---|
| D1 | E00, E01 (Compose, RustFS, **Polaris round-trip**) | E02 schemas, H3 helpers | Iceberg table written + read through Polaris; Trino sees it |
| D2 | E01 Trino/Postgres/Dagster; start E04 | E02 finish, E03 fields + coffee config | 3 field polygons in Postgres + `core.field` |
| D3 | E04 finish, E05 NDVI | E06 ET0 + water balance | `core.weather_observation`, `core.vegetation_observation` populated |
| D4 | E08 asset graph + Postgres sync | E06 finish, E07 products + risk | `products.*` materialized end-to-end by Dagster |
| D5 | E11 map + field state card | E09 API + MCP, E10 agent | Both question types answered via API |
| D6 | E11 Water Optimizer + Copilot UI + Why view | E10 eval set, E12 snapshot | Full demo runs on frozen snapshot |
| D7 | E12 rehearsal, backup video | E12 pitch, README, bug fixes | Two clean rehearsals under 5 minutes |

### One-developer cut list (apply in order)

1. Drop Trino from the demo path; keep it in Compose for one SQL slide (E01 S1.4 → Could).
2. Use Open-Meteo's ET0 directly; skip own Penman-Monteith (E06 S6.1 → Could, keep cross-check off).
3. NDVI for the 3 fields as a one-off precomputed run, not a Dagster asset (E05 S5.3 → manual script).
4. Single-page web app without the provenance drawer (E11 S11.5 → Could).
5. Reduce agent eval set to 5 questions (E10 S10.4).

---

## 8. Story format

Every story in the epic files follows this layout:

```markdown
### S<epic>.<n> — Title  `Priority` `Estimate`
As a <user>, I want <capability>, so that <outcome>.

**Acceptance criteria**
- Given … when … then …

**Subtasks**
- [ ] …
```

---

## 9. Definition of Done (all stories)

- Code lives in the location defined by the repo layout (E00 S0.1).
- Domain logic has unit tests; `pytest` passes in CI.
- Every product row carries `model_version`, `quality_score` and lineage (PRD §30–31, §39).
- No secrets in the repo; config is read from `.env`.
- Any scientific parameter used has a citation in `packages/earthflow-agriculture/config/coffee.yaml` or the relevant doc.
