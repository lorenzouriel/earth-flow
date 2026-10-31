# E11 — Web App (Map, Water Optimizer, Farm Copilot)

**Priority:** Must · **Estimate:** 1.5d · **Day:** D5–6 · **Owner:** Dev A
**References:** PRD §26–29, §35 (Frontend); Architecture §1

## Goal

One Next.js + MapLibre app with two clearly separate applications, **Water Optimizer** and **Farm Copilot**, visibly reading the same field products. This is the "two apps, one platform" moment the judges must see (PRD §27, §42.3).

## Layout

```text
┌──────────────────────────────────────────────────────────────┐
│ Earth Flow · São Paulo        [Water Optimizer] [Farm Copilot]│
├───────────────────────────────┬──────────────────────────────┤
│                               │ Field card                    │
│        MapLibre map           │ Crop: Coffee                  │
│     (3 field polygons,        │ Vegetation health 78          │
│      NDVI H3 layer)           │ Water status 61               │
│                               │ Heat risk High · Drought Med  │
│                               ├──────────────────────────────┤
│                               │ Active app panel              │
│                               │ (Optimizer or Copilot)        │
└───────────────────────────────┴──────────────────────────────┘
        "Powered by the same Earth Flow products" ribbon
```

---

### S11.1 — App scaffold & API client  `Must` `S`
As a developer, I want the Next.js app wired to the API, so that every screen uses typed data.

**Acceptance criteria**
- Next.js (App Router, TypeScript) in `apps/web`; TypeScript types generated from the API's OpenAPI schema.
- API base URL from env; loading and error states for every fetch.

**Subtasks**
- [ ] Scaffold, Tailwind, lint
- [ ] `openapi-typescript` client generation script
- [ ] Add web job to CI (E00 S0.3)

### S11.2 — Map with fields  `Must` `M`
As a demo user, I want a map of São Paulo with the coffee fields, so that I can pick one (PRD §27 steps 1–2).

**Acceptance criteria**
- MapLibre GL map centered on São Paulo state with a satellite or terrain basemap whose license allows the demo.
- 3 field polygons from `GET /v1/fields`, colored by the top risk level; click to select and fly to the field.
- NDVI H3 cell layer for the selected field (Should).

**Subtasks**
- [ ] Choose basemap tiles; check license / attribution
- [ ] Field layer + selection state
- [ ] H3 layer via `h3-js` → GeoJSON

### S11.3 — Field state card  `Must` `S`
As a farmer, I want a plain-language summary of my field, so that I see decisions, not raw indices (PRD §27 step 3, §28).

**Acceptance criteria**
- Shows crop, crop stage, vegetation health, water status, heat risk and drought risk from `/state`.
- One-sentence plain-language line per metric (e.g. "Vegetation slightly below normal for this season").
- "Expert view" toggle reveals raw NDVI, ET0 and depletion in mm.

**Subtasks**
- [ ] Card component
- [ ] Copy text rules (template-based, not LLM)

### S11.4 — Water Optimizer panel  `Must` `M`
As a farmer, I want to see whether and how much to irrigate this week, so that I save water (PRD §26 A, §27 step 5).

**Acceptance criteria**
- Shows: recent rainfall (7/30 d), forecast rainfall, crop water demand, soil-water condition, recommended irrigation (mm + liters), timing, avoided liters vs the baseline (baseline stated on screen), confidence.
- 30-day history + 16-day forecast chart: rainfall bars, ETc line, soil water area; observed and forecast parts visually distinct.

**Subtasks**
- [ ] Panel component calling `POST /irrigation` and `/water-balance`
- [ ] Chart (Recharts or similar)
- [ ] Number formatting in PT-BR (`1.400.000 litros`)

### S11.5 — "Why?" and provenance drawer  `Should` `M`
As a scientist, I want to see why a value is what it is and where it came from, so that I trust it (PRD §29, §30).

**Acceptance criteria**
- Every risk and recommendation has a "Por quê?" button opening the evidence table.
- A provenance tab renders the lineage (model version, datasets, generated at).
- Badges distinguish **Observado**, **Previsão**, **Modelo** and **Explicação IA** (PRD §29).

**Subtasks**
- [ ] Drawer component
- [ ] Badge component used across card, optimizer and copilot

### S11.6 — Farm Copilot chat  `Must` `M`
As a farmer, I want to ask questions about the selected field, so that I get a ranked risk answer (PRD §26 B, §27 step 6).

**Acceptance criteria**
- Chat panel posting to `/v1/copilot/ask` with streaming display.
- Suggested prompts: "Preciso irrigar esta semana?", "Qual o maior risco para essa plantação?", "O que devo observar esta semana?".
- Structured answers render as risk cards with evidence, actions, confidence and sources; free text is marked **Explicação IA**.

**Subtasks**
- [ ] Chat UI with SSE
- [ ] Risk card renderer for the E10 S10.3 schema
- [ ] Suggested prompt chips

### S11.7 — "Same platform" indicator  `Should` `S`
As a judge, I want to see that both apps use the same data products, so that the platform thesis is obvious (PRD §27 last line, §42.3).

**Acceptance criteria**
- A small panel lists the products and versions each app read (e.g. `products.field_state@field_state_v0.1`, `features.water_balance@water_balance_v0.1`), highlighting the shared ones.

**Subtasks**
- [ ] Collect product refs from response lineage
- [ ] Ribbon / panel component
