# News Knowledge Graph

Nightly pipeline that ingests news articles, extracts and resolves named entities
against Wikidata, builds a knowledge graph, and serves an explorer web app plus a
"ask the graph" (GraphRAG) interface.

Status: pre-implementation. Read docs/ROADMAP.md for the current phase.
Agents: read AGENTS.md first.

## Components
| Dir | What |
|---|---|
| pipeline/ | Python ingestion, NER, entity resolution, extraction, loaders |
| airflow/ | DAG definitions that orchestrate pipeline tasks |
| api/ | FastAPI service (search, graph, trending, ask) |
| web/ | Next.js 15 + TypeScript explorer UI |
| dbt/ | dbt project (DuckDB) over the Parquet lake |
| infra/ | Terraform (AWS) |
| evals/ | GraphRAG and entity-resolution evaluation sets and runners |
| db/ | Postgres migrations (canonical schema) |
| docs/ | Design docs, decisions, manual setup |

## Quickstart (local)
1. Complete docs/MANUAL_SETUP.md and fill .env from .env.example
2. `docker compose up -d postgres redis neo4j` (local dev stack)
3. `make migrate && make ingest-sample`
4. `make api` and `make web`
