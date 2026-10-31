# E09 — FastAPI & MCP Server

**Priority:** Must · **Estimate:** 1d · **Day:** D4–5 · **Owner:** Dev B
**References:** PRD §13, §32, §33; Architecture §1, §27; README decision D12

## Goal

A stable REST API (OpenAPI) and an MCP tool server over the same product layer. **The tools are the architecture** (Architecture §27): the web app and the agent call the same functions.

```text
apps/api (FastAPI) ──┐
                     ├──► earthflow_core.services (read serving.* / compute) ──► Postgres
earthflow-mcp ───────┘
```

---

### S9.1 — Shared service layer  `Must` `M`
As a developer, I want one Python service module used by both REST and MCP, so that results can never differ between the app and the agent.

**Acceptance criteria**
- `services.get_field_state(field_id, date)`, `get_weather(field_id, period)`, `get_weather_forecast(field_id)`, `get_vegetation(field_id, period)`, `get_water_balance(field_id, period)`, `get_field_risks(field_id)`, `calculate_irrigation(field_id)`, `get_crop_guidance(crop, topic)`.
- Every response includes `quality`, `lineage` and a `kind` per value (`observed | reanalysis | forecast | model`).
- The "today" date comes from `DEMO_SNAPSHOT_DATE` when set.

**Subtasks**
- [ ] Service module + Pydantic response models (reuse E02)
- [ ] Postgres repository functions
- [ ] Tests against a seeded test database or fixtures

### S9.2 — REST endpoints  `Must` `M`
As an app developer, I want the PRD §32 endpoints, so that applications consume products, not raw datasets.

**Acceptance criteria**
- `GET /v1/fields` (GeoJSON FeatureCollection)
- `GET /v1/fields/{id}/state`
- `GET /v1/fields/{id}/weather?from&to`
- `GET /v1/fields/{id}/vegetation?from&to`
- `GET /v1/fields/{id}/water-balance?from&to`
- `GET /v1/fields/{id}/risks`
- `POST /v1/fields/{id}/irrigation` (returns the recommendation; optional body overrides the baseline schedule)
- OpenAPI docs at `/docs`; 404 for unknown field; CORS for the web app.

**Subtasks**
- [ ] Routers in `apps/api/earthflow_api/routers/`
- [ ] API tests with httpx `TestClient`

### S9.3 — Crop guidance content  `Should` `S`
As the Copilot, I want short, sourced guidance per risk, so that recommended actions are grounded in documents, not invented (PRD §34, minimal version).

**Acceptance criteria**
- `packages/earthflow-agriculture/guidance/coffee/*.md` covering water stress, heat stress, drought and vegetation decline; each has a source citation.
- `get_crop_guidance(crop, topic)` returns the matching text + source. Keyword lookup only; no vector search in the MVP.

**Subtasks**
- [ ] Write 4–6 short guidance notes from Embrapa / IAC material
- [ ] Lookup function + test

### S9.4 — earthflow-mcp server  `Must` `M`
As an AI agent, I want restricted, well-described environmental tools, so that I never need raw database access (PRD §5.5, §33).

**Acceptance criteria**
- MCP server (official MCP Python SDK) exposing 7 tools: `get_field_state`, `get_weather_forecast`, `get_water_balance`, `get_vegetation_health`, `get_field_risks`, `calculate_irrigation`, `get_crop_guidance`.
- Each tool has a clear description and typed input schema; outputs are JSON that includes units, quality and lineage.
- Missing data returns an explicit `unavailable` result with a reason, never an empty success (PRD §33).
- Runs over stdio (for the agent) and streamable HTTP (for external clients).

**Subtasks**
- [ ] `packages/earthflow-mcp/src/earthflow_mcp/server.py` calling the service layer
- [ ] Tool descriptions reviewed for agent clarity
- [ ] Test with MCP Inspector; record a screenshot for the pitch

### S9.5 — Analyst SQL path  `Could` `S`
As a researcher, I want to query the same products with SQL, so that the demo shows non-API consumers (PRD §5.2).

**Acceptance criteria**
- `docs/demo/queries.sql` runs in Trino against `iceberg.products.*` and returns the same numbers as the API for the snapshot date.

**Subtasks**
- [ ] 3 demo queries
- [ ] Parity check script
