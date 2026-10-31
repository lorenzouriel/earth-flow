# E00 — Repo Foundation

**Priority:** Must · **Estimate:** 0.5d · **Day:** D1 · **Owner:** Dev A
**References:** Architecture §26, §27, §29; PRD §36

## Goal

A monorepo that both developers can clone, install and test in minutes, with the layout from Architecture §29 so domain logic stays portable.

## Repo layout (MVP subset of Architecture §29)

```text
earth-flow/
├── packages/
│   ├── earthflow-core/          # schemas, provenance, quality
│   ├── earthflow-geo/           # H3, geometry, STAC helpers
│   ├── earthflow-agriculture/   # coffee config, crop stage, risk scoring
│   ├── earthflow-water/         # ET0, water balance, irrigation
│   └── earthflow-mcp/           # MCP tool server
├── ingestion/
│   ├── open_meteo/
│   └── sentinel/
├── products/                    # product builders (call packages)
│   ├── field_state/
│   ├── irrigation/
│   └── field_risk/
├── apps/
│   ├── api/                     # FastAPI + Copilot endpoint
│   └── web/                     # Next.js + MapLibre
├── platforms/
│   └── local/                   # docker compose, Polaris bootstrap, Trino config, Dagster defs
├── data/
│   ├── seed/                    # field polygons, soil profiles
│   └── snapshot/                # frozen demo data (E12)
└── docs/
```

---

### S0.1 — Monorepo skeleton  `Must` `S`
As a developer, I want the agreed folder layout and workspace config in place, so that code lands in the right place from the first commit.

**Acceptance criteria**
- Given a fresh clone, when I run `uv sync`, then all Python packages install as editable workspace members.
- Given a fresh clone, when I run `pnpm install` in `apps/web`, then dependencies install.
- The layout matches the tree above.

**Subtasks**
- [ ] `git init`; add `.gitignore` (Python, Node, `.env`, `data/raw/`)
- [ ] Root `pyproject.toml` with a uv workspace listing `packages/*`, `ingestion/*`, `products/*`, `apps/api`
- [ ] One `pyproject.toml` per package with `src/` layout
- [ ] Pin Python version (`.python-version`)
- [ ] `apps/web` placeholder (filled in E11)
- [ ] Root `README.md` with quick start (filled further in E12)

### S0.2 — Quality tooling  `Must` `S`
As a developer, I want lint, format and test commands that work the same everywhere, so that the two developers don't fight over style.

**Acceptance criteria**
- `make lint`, `make test`, `make fmt` (or `just` equivalents) run from the repo root.
- `pytest` runs across all packages and discovers an empty smoke test per package.

**Subtasks**
- [ ] Ruff (lint + format) config in root `pyproject.toml`
- [ ] pytest config with per-package `tests/`
- [ ] `Makefile` / `justfile` with `lint`, `fmt`, `test`, `up`, `down`
- [ ] Pre-commit hook running Ruff (optional)

### S0.3 — CI pipeline  `Should` `S`
As a developer, I want GitHub Actions to run lint and tests on every push, so that breakages show up before the demo.

**Acceptance criteria**
- Given a push, CI runs `ruff check` and `pytest` and reports status.
- CI does not need Docker services (domain tests use fixtures).

**Subtasks**
- [ ] `.github/workflows/ci.yml` with uv setup + cache
- [ ] Web lint/typecheck job (added once E11 starts)

### S0.4 — Configuration & secrets  `Must` `S`
As a developer, I want one place for configuration, so that API keys and endpoints are never hard-coded.

**Acceptance criteria**
- `.env.example` lists every variable; `.env` is git-ignored.
- `earthflow-core` exposes a typed settings object (pydantic-settings).

**Subtasks**
- [ ] `.env.example`: `ANTHROPIC_API_KEY`, `POLARIS_*`, `S3_*` (RustFS), `POSTGRES_*`, `TRINO_*`, `DEMO_SNAPSHOT_DATE`
- [ ] `earthflow_core.settings.Settings` class
