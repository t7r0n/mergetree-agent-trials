# ClickHouse AgentBench Local

Offline SQL-agent safety and performance benchmark for ClickHouse MCP-style access.

This is a local-first, synthetic-data prototype inspired by a company-specific project plan for **ClickHouse**. It is built to demonstrate the engineering shape of `AgentBench` without private data, credentials, external APIs, or hosted services.

## Why it matters

AI assistants connected to analytical databases need proof they generate safe, bounded, performant ClickHouse SQL.

## What it does

- Generates deterministic synthetic `sql task` scenarios.
- Scores each scenario against domain-specific quality gates.
- Produces evidence-backed findings for realistic failure modes.
- Writes a static dashboard, JSON reports, benchmark output, and a portable demo pack.
- Exposes a JSONL tool loop for local agent integration.

## Metrics

- `query_correctness`
- `cost_bound`
- `readonly_safety`
- `schema_grounding`

## Failure modes

- `full_scan_explosion`
- `write_attempt`
- `array_syntax_error`
- `ungrounded_join`

## Quickstart

```bash
uv sync --extra dev
uv run ch-agentbench init-demo --force
uv run ch-agentbench run-suite
uv run ch-agentbench verify
uv run ch-agentbench dashboard
uv run ch-agentbench benchmark --iterations 100
uv run ch-agentbench export-demo-pack
```

## Expected outputs

- `data/scenarios.json`
- `outputs/summary.json`
- `outputs/reports.json`
- `outputs/evidence_pack.md`
- `outputs/dashboard.html`
- `outputs/benchmark.json`
- `outputs/demo-pack.zip`

## Validation

```bash
uv run ruff check .
uv run pytest -q
uv run ch-agentbench run-suite
uv run ch-agentbench verify
uv run ch-agentbench benchmark --iterations 100
```

## Demo hook

A benchmark card shows the exact ClickHouse-specific SQL construct an agent used correctly or dangerously.
