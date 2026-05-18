# ClickHouse AgentBench Local

Offline SQL-agent safety and performance benchmark for ClickHouse MCP-style access.

`ClickHouse AgentBench Local` is framed as an engineering instrument rather than a pitch deck: generate cases, break them, score them, and inspect the evidence.

## Decision surface

Offline SQL-agent safety and performance benchmark for ClickHouse MCP-style access.

## Evaluator shape

- Creates a clean/degraded `sql task` corpus sized for local iteration: 200 cases.
- Uses `query_correctness`, `cost_bound`, `readonly_safety`, and `schema_grounding` to explain why a run passed, failed, or needs inspection.
- Turns `full_scan_explosion`, `write_attempt`, `array_syntax_error`, and `ungrounded_join` into executable cases with visible evidence trails.
- Carries the same `AgentBench` result through JSON, dashboard, benchmark, and demo-pack outputs.

## Quick path

```bash
uv sync --extra dev
uv run ch-agentbench init-demo --force
uv run ch-agentbench run-suite
uv run ch-agentbench verify
uv run ch-agentbench dashboard
uv run ch-agentbench benchmark --iterations 100
uv run ch-agentbench export-demo-pack
```

## Materialized results

- `data/scenarios.json`
- `outputs/summary.json`
- `outputs/reports.json`
- `outputs/evidence_pack.md`
- `outputs/dashboard.html`
- `outputs/benchmark.json`
- `outputs/demo-pack.zip`

## Acceptance checks

```bash
uv run ruff check .
uv run pytest -q
uv run ch-agentbench run-suite
uv run ch-agentbench verify
uv run ch-agentbench benchmark --iterations 100
```

## Repo boundary

`ClickHouse AgentBench Local` is built for local reproduction: deterministic inputs enter the run, deterministic evidence comes out, and private data stays outside the repo.
