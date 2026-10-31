# E12 — Demo Hardening & Narrative

**Priority:** Must · **Estimate:** 1d · **Day:** D6–7 · **Owner:** Dev A + Dev B
**References:** PRD §27, §41–42, §45–46; README decision D11

## Goal

The demo works every time, offline if needed, and tells the PRD §46 story in under 5 minutes. The core message must land in the first minute.

---

### S12.1 — Frozen demo snapshot  `Must` `M`
As the presenter, I want the whole demo to run on a fixed date with preloaded data, so that live APIs and cloud cover can't break it (decision D11).

**Acceptance criteria**
- Choose `DEMO_SNAPSHOT_DATE` where at least one field has a forecast rain event (for the "delay irrigation" story) and one has a clear heat or water risk (for the Copilot story).
- `make demo-load` restores Iceberg tables + Postgres serving tables from `data/snapshot/` on a clean machine, with no network calls except the Anthropic API.
- `make demo` = `make up` + `make demo-load` + start API and web.

**Subtasks**
- [ ] Scan the backfill for a good date per field
- [ ] Export / import scripts for the snapshot (Iceberg data via pyiceberg, Postgres via `pg_dump`)
- [ ] Clean-machine test

### S12.2 — Demo script & story  `Must` `M`
As the presenter, I want a rehearsed script that follows PRD §27 and §46, so that judges understand the thesis in a minute.

**Acceptance criteria**
- `docs/demo/script.md`:
  - Problem (15 s)
  - Map → field card (30 s)
  - Water Optimizer: "Preciso irrigar?" (60 s)
  - Copilot: "Qual o maior risco?" (60 s)
  - "Why?" + provenance (30 s)
  - Same-platform reveal + architecture slide (45 s)
  - Beyond agriculture (30 s)
- Each beat maps to a PRD §42 success criterion.

**Subtasks**
- [ ] Write script
- [ ] Two timed rehearsals; adjust

### S12.3 — Pitch slides  `Must` `M`
As the presenter, I want a short deck, so that the architecture and impact are clear around the live demo.

**Acceptance criteria**
- About 6 slides:
  - Problem
  - Earth Flow thesis
  - Live demo placeholder
  - Architecture (layers + Local/OSS stack: RustFS, Polaris, Iceberg, Trino, Dagster, FastAPI, MCP)
  - Impact: liters avoided per field per week, extrapolated with stated assumptions
  - Roadmap (PRD §43 phases)
- Ends with "We are building the reusable intelligence layer beneath climate applications." (PRD §46)

**Subtasks**
- [ ] Architecture diagram
- [ ] Impact numbers from the snapshot, with assumptions on the slide

### S12.4 — Backup recording  `Must` `S`
As the presenter, I want a recorded video of the full demo, so that there's a fallback if anything fails live.

**Acceptance criteria**
- 3–4 minute screen recording of the full flow at 1080p, stored locally and in the cloud.

**Subtasks**
- [ ] Record after the final rehearsal

### S12.5 — Repo README & model docs  `Should` `S`
As a judge or future contributor, I want a clear README, so that the project reads as open infrastructure (PRD §40).

**Acceptance criteria**
- Root README: positioning line (PRD §45), architecture diagram, quick start (`make demo`), data sources and licenses, model docs links, known limitations.
- `docs/models/` contains water balance, field risk and vegetation docs (assumptions, equations, datasets, limitations — PRD §39).

**Subtasks**
- [ ] README
- [ ] Data source attribution: Open-Meteo, Copernicus Sentinel-2, FAO-56, MapBiomas

### S12.6 — Bug bash  `Must` `M`
As the team, I want a dedicated block for fixing what the rehearsals expose, so that polish doesn't eat into the demo.

**Acceptance criteria**
- All issues found in rehearsals are triaged; Must-level ones are fixed or have a workaround in the script.

**Subtasks**
- [ ] Rehearsal issue list
- [ ] Fix / workaround each one
