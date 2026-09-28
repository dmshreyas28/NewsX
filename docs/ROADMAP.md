# Roadmap

Rule: phases are sequential; tasks within a phase are ordered. Do not start a phase
until the previous phase's exit criteria are met. Each task is sized for one agent session.

## Phase 0 — Foundations
- T0.1 Repo scaffold: dirs per README, Makefile, .env.example, .gitignore, pre-commit (ruff, prettier)
- T0.2 docker-compose.yml: postgres (pgvector), neo4j, redis; healthchecks
- T0.3 CI (GitHub Actions): lint, type-check, test on PR; cache deps
- T0.4 Config module (pydantic-settings) + structured logging util
Exit: `make check` green in CI on an empty-but-wired repo; `docker compose up` healthy.

## Phase 1 — Core pipeline (correctness first)
- T1.1 db/migrations: core schema from DATA_MODEL.md; migration runner; tests
- T1.2 Ingest: RSS + GDELT connectors, canonical URL, dedup, text extraction, rate limiting
- T1.3 Lake writer (Parquet, partition overwrite) + pandera schemas
- T1.4 NER stage + mentions persistence
- T1.5 Entity resolution per ENTITY_RESOLUTION.md + evals/er runner and labeled set (human task: labeling)
- T1.6 LLM extraction per PROMPTS.md + llm_calls logging + cache
- T1.7 Neo4j projector (idempotent) + `rebuild-graph`
- T1.8 Data-quality gates + pipeline_runs bookkeeping
- T1.9 CLI: `python -m pipeline.run --date YYYY-MM-DD [--stage ...]`
- T1.10 Dockerfile(s) for pipeline; image builds in CI
Exit: one-day run end to end locally in Docker; ER F1 reported; tests >= 80% on pipeline/.

## Phase 2 — API, cache, web, instrumentation
- T2.1 FastAPI skeleton, health, config, OTel instrumentation, OpenAPI export
- T2.2 Endpoints: /entities/search, /entities/{id}, /entities/{id}/neighbors, /graph/path,
  /trending, /articles/{id}
- T2.3 Redis caching layer with TTLs + cache-hit metrics
- T2.4 Auth-free rate limiting, pagination, error model
- T2.5 Web: generated TS client, search page, entity page, force-graph explorer
- T2.6 Playwright e2e for search -> entity -> graph flow; API contract tests
Exit: full local stack demo; e2e tests green in CI.

## Phase 3 — Orchestration
- T3.1 Airflow in compose (LocalExecutor); DAG `nightly_news_kg` wrapping pipeline tasks
- T3.2 Retries, SLAs, failure alerts, backfill/catchup validated for 7 past days
- T3.3 DAG tests (import + structure) in CI
Exit: scheduled local run succeeds; a deliberately failed task recovers on retry.

## Phase 4 — Cloud and IaC
- T4.1 Terraform: remote state, S3 lake bucket (lifecycle rules), ECR, IAM (least privilege)
- T4.2 Deploy API (runtime per ADR-006) + secrets via SSM/Secrets Manager
- T4.3 Pipeline writes lake to S3; production scheduled run (per ADR-007)
- T4.4 CD workflow: build, push, terraform plan on PR / apply on main with approval
- T4.5 Cost guardrails: AWS Budgets alert, teardown doc
Exit: public API URL serving real data; `terraform destroy` documented and tested.

## Phase 5 — Analytics engineering
- T5.1 dbt project (dbt-duckdb) reading lake Parquet; sources + staging models
- T5.2 Marts from DATA_MODEL.md with schema + data tests
- T5.3 Export marts to Postgres `analytics` schema; /trending reads from it
- T5.4 dbt build in CI against fixture Parquet
Exit: trending endpoint powered by dbt marts; docs generated (dbt docs).

## Phase 6 — GraphRAG + evals
- T6.1 Chunk + embed articles (embedding model per config), HNSW index
- T6.2 Retriever: vector search + graph expansion (1-2 hops) + rerank
- T6.3 /ask endpoint with citations and refusal behavior
- T6.4 evals/rag: 60+ questions with gold article IDs and gold facts (human-labeled);
  metrics: retrieval recall@k, citation precision, faithfulness (LLM judge, spot-checked by hand)
- T6.5 Web "Ask" UI showing citations linked to source URLs
Exit: metrics reported and tracked over at least 3 iterations.

## Phase 7 — Hardening and portfolio
- T7.1 Grafana dashboards + alerts; runbook
- T7.2 Load test API (k6 or locust), record p95
- T7.3 README with architecture diagram, screenshots, metrics table
- T7.4 Write-up: 5 key decisions, what failed, what you'd change
