# Architecture

## Goal
Turn ~500 news articles/day into a queryable knowledge graph of entities, events,
and relationships, with an explorer UI and a question-answering interface.

## Data flow
1. Ingest: RSS (feedparser) + GDELT -> article metadata + fetched text (trafilatura).
   Dedup by canonical URL and near-duplicate title/content hash.
2. Raw layer: write immutable daily Parquet to the lake (S3 in cloud, ./data/lake locally):
   lake/raw/articles/date=YYYY-MM-DD/*.parquet
3. NER: spaCy (transformer model configurable; small model for CI) -> mentions.
4. Entity resolution: mentions -> Wikidata QIDs (see ENTITY_RESOLUTION.md).
5. Event/relation extraction: LLM with JSON-schema output (see PROMPTS.md).
6. Load:
   - Postgres (system of record): articles, mentions, entities, events, embeddings (pgvector)
   - Neo4j (graph projection): rebuilt/upserted from Postgres, never written independently
7. Analytics: dbt (DuckDB) over lake Parquet -> marts (trending, co-occurrence, entity daily counts)
8. Serve: FastAPI reads Postgres, Neo4j, Redis cache; Next.js consumes generated client.
9. Ask: GraphRAG endpoint (vector retrieval + graph expansion + LLM answer with citations).

## Key principles
- Postgres is the single source of truth. Neo4j is a derived projection and can be
  dropped and rebuilt from Postgres at any time (`make rebuild-graph`).
- The lake is append-only; reruns for a date overwrite that date's partition atomically.
- Orchestration (Airflow) calls the same pipeline entrypoints as the CLI. No logic in DAG files.
- Every stage has a data-quality gate (pandera schemas + row-count/null-rate checks).
  A failed gate fails the task and does not load downstream.

## Services and environments
- local: docker compose (postgres+pgvector, neo4j, redis, airflow, api, web)
- cloud (AWS, Terraform): S3 lake, ECR, API runtime (see ADR-006), Secrets Manager/SSM,
  Neon Postgres and Aura Neo4j stay external managed services initially.
- Web on Vercel, API base URL from env.

## Observability
OpenTelemetry traces + metrics from api and pipeline; export to Grafana Cloud free tier.
Key metrics: articles ingested, mentions/article, ER link rate, NIL rate, LLM cost/day,
task duration, API p95 latency, cache hit rate.

## Non-goals
Real-time streaming, multi-language support, user accounts (v1), serving article text.
