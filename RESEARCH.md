# Research And Plan Review

Company: ClickHouse
Project: `AgentBench`

## Refined Thesis

AI assistants connected to analytical databases need proof they generate safe, bounded, performant ClickHouse SQL.

The implementation is intentionally local and synthetic, but the test harness is shaped around the real operating question: can the proposed artifact create evidence a founder, CTO, or product leader would immediately recognize as useful?

## Fresh Sources Checked

- https://github.com/ClickHouse/mcp-clickhouse
- https://clickhouse.com/
- https://altinity.com/wp-content/uploads/2026/04/OAuth-in-Altinity-Builds-Making-ClickHouse-Safe-for-AI-2026-04-28.pdf

## Plan Excerpt Used

## The Gap

ClickHouse has shipped the MCP server and Ask AI in beta, and they've made the right marketing claim — *the database for AI agents*. But the **proof** they need to back that claim with — *"AI agents emit queries 100× faster and ClickHouse handles it where Snowflake/BigQuery/Postgres don't"* — does not yet exist as a public artifact. [ClickBench](https://github.com/ClickHouse/ClickBench) measures *human-pattern* analytical queries (TPC-H-ish: ad-hoc aggregations over hits/logs). Nobody, including ClickHouse, has published a benchmark for **the agentic workload shape**: bursty schema introspection + many small parameterised `SELECT`s with high cardinality `WHERE` + retries from a tool-calling loop + MCP-protocol overhead. That benchmark is the artifact that converts Katz's quote into a chart competitors can't argue with — and it's the natural extension of ClickBench, which the team has already established as the credible measuring stick.

## The Project — `AgentBench` (working title: `ClickBench-Agents`)

> The first public benchmark for **agentic analytical workloads** — driven by a real LLM tool-calling loop over MCP, scoring databases on the query shapes AI agents actually emit.

1. **What it is.** A reproducible benchmark harness with a fixed dataset (1B-row + 100M-row variants of a realistic e-commerce schema), a fixed set of 50 *agentic tasks* expressed in natural language ("which products had > 2σ return-rate spikes in the last 6 hours?"), and a fixed agent runner that uses a real LLM (Claude / GPT-4-class) over MCP to plan, introspect, and answer. Outputs: per-task **time-to-first-correct-answer**, **p95 per-SQL-step latency**, **MCP roundtrip count**, **tokens-on-schema** (how much schema fits in context), **dollar-cost-per-task**.
2. **Why it solves the gap.** ClickBench measures *human-written queries*. AgentBench measures *agent-emitted query traces*. The two are different distributions: agents bias toward schema-introspection bursts (`SHOW`, `DESCRIBE`, sample reads), short paginated probes (`LIMIT 5`), and chained refinements (a `WHERE` filter that adds three more clauses across three turns). The benchmark fingerprints ClickHouse's actual differentiators — fast `system.columns` introspection, projection pushdown on `LIMIT`, FINAL / ARRAY JOIN — that don't show up in TPC-H-style tests but absolutely show up under an LLM. It is the **chart** ClickHouse needs.
3. **The "wow" moment.** A live leaderboard at `agentbench.run` showing ClickHouse Cloud, ClickHouse OSS, Snowflake, BigQuery, DuckDB, Postgres, and *chdb* running the **same** Claude-driven agentic task suite, with the per-task transcript, MCP wire log, and SQL plan all exportable. ClickHouse wins the "time-to-first-correct-answer" and "tokens-on-schema" columns by a wide margin — and you can *click into the trace* to see why.

## Prototype Plan (the shippable demo)

**Surface a user invokes:**

```bash
agentbench run \
  --dataset ecommerce-1b \
  --target clickhouse-cloud \
  --agent-llm claude-sonnet-4-5 \
  --tasks tasks.yaml \
  --seeds 30 \
  --out results/clickhouse-cloud.json
agentbench leaderboard publish results/*.json
```

**Five representative tasks (the demo subset):**

1. *"Which 5 SKUs had the largest week-over-week drop in conversion?"* — joins + window over 1B rows.
2. *"Show me an example row from every table whose name contains `order`."* — heavy schema-introspection burst, classic ClickHouse win.
3. *"What's the p95 add-to-cart latency for users in the EU yesterday, by device?"* — high-cardinality `WHERE` + `quantileTDigest`.
4. *"Refine: now exclude bots; now further restrict to mobile-Safari."* — multi-turn chained refinement; tests CTE / sub-query caching.
5. *"Materialize a fact table that joins orders + sessions for the last 7 days; query it."* — `ARRAY JOIN` + projection.

**Expected output / screen:** a markdown table + a live HTML leaderboard at `agentbench.run`, plus per-task transcript JSON viewable in a browser.

**What we measure (to prove it works):**
- p95 time-to-first-correct-answer per task.
- p95 SQL step latency.
- MCP roundtrip count per task.
- Tokens of schema context consumed per task.
- Faithfulness score (LLM-judge, 0-1).
- $ per task (sum of LLM tokens + DB-side query cost where measurable).


## Build Acceptance Criteria

- Deterministic local fixtures.
- Domain-specific metrics and failure modes.
- Passing unit tests.
- Passing CLI verifier.
- Static dashboard generated locally.
- Benchmark output under the project `outputs/` folder.
- Public-safe README: no founder emails, no private outreach text, no credentials.
