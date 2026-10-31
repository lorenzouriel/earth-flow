# E10 — Farm Copilot Agent

**Priority:** Must · **Estimate:** 1d · **Day:** D5 · **Owner:** Dev B
**References:** PRD §3.4, §12, §15, §26 B, §27 steps 4–6, §29, §33; Architecture §25, §27; README decision D10

## Goal

A Portuguese-speaking agent, powered by Claude via the Anthropic API, that answers field questions using **only** the `earthflow-mcp` tools.

The deterministic models calculate; the LLM explains (PRD §3.4). Numbers in the answer must match tool output exactly.

```text
User question (PT-BR)
   ↓
Copilot agent (PydanticAI + Claude)
   ↓ MCP (stdio)
earthflow-mcp tools → service layer → products
   ↓
Structured answer: risks ranked · evidence · actions · confidence · sources
```

---

### S10.1 — Agent runtime  `Must` `M`
As a farmer, I want to ask questions in Portuguese about my field, so that I get decisions instead of raw data (PRD §28).

**Acceptance criteria**
- PydanticAI agent using the Anthropic provider. The model is configurable; default is the latest Claude model, with a faster model allowed if latency is too high for the demo.
- Connects to `earthflow-mcp` as an MCP server; no other tools or data access.
- `POST /v1/copilot/ask {field_id, question}` in `apps/api` returns the structured answer; supports streaming (SSE) for the UI.
- Answers "Preciso irrigar esta semana?" by calling `calculate_irrigation`, and "Qual o maior risco para essa plantação?" by calling `get_field_risks` (+ `get_crop_guidance`).

**Subtasks**
- [ ] `apps/api/earthflow_api/copilot/agent.py`
- [ ] MCP client wiring (stdio subprocess)
- [ ] SSE endpoint
- [ ] Timeout and error handling (tool error → honest message)

### S10.2 — System prompt & guardrails  `Must` `M`
As a scientist, I want the agent to never invent measurements, so that the product stays credible (PRD §12, §33).

**Acceptance criteria**
- Prompt rules:
  - Every number must come from a tool result in this conversation.
  - If data is unavailable, say so.
  - Separate observed, forecast and model values from your own explanation.
  - Answer in the user's language (PT-BR default).
  - Never present uncertain estimates as facts; always state confidence.
  - Recommended actions come from `get_crop_guidance` or are clearly framed as general advice.
- Prompt lives in a versioned file `copilot_prompt_v0.1.md`.

**Subtasks**
- [ ] Write prompt
- [ ] Post-check: extract numbers from the answer and verify each appears in tool outputs; flag mismatches in logs (Should)

### S10.3 — Structured answer schema  `Must` `S`
As the web app, I want a predictable answer shape, so that the UI can render risks, evidence and the "Why?" view (PRD §26 B, §29).

**Acceptance criteria**
- Output model with:
  - `summary` (text)
  - `risks[]` — `type`, `level`, `score`, `evidence[]`, `confidence`
  - `recommendation` (optional irrigation block)
  - `actions[]` — each with a source
  - `sources[]` — lineage refs
  - `generated_by: "ai_explanation"`
- The UI labels AI text separately from model outputs.

**Subtasks**
- [ ] Pydantic output type for the agent
- [ ] Map tool lineage into `sources[]`

### S10.4 — Evaluation set  `Should` `M`
As the team, I want a small regression set of questions, so that prompt changes don't break the demo.

**Acceptance criteria**
- 10 questions (PT-BR + 2 in English) × 3 fields, run by `make eval-copilot`.
- Checks:
  - The right tool was called
  - Every number in the answer appears in tool output
  - A missing-data question gets an "unavailable" answer
  - The language matches the question

**Subtasks**
- [ ] Question set YAML
- [ ] Eval runner script
- [ ] Record pass rate in `docs/demo/eval.md`
