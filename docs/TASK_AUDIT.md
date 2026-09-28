# Task Audit

Audit scope: `docs/tasks/phase-0.md` through `docs/tasks/phase-7.md` compared with
the base documents created in Step 1. This report does not modify the task plans.

## 1. References in task files

Legend: **yes** means the reference exists in the base docs; **no** means it does
not; **mismatch** means a related item exists but the task plan changes, extends,
or conflicts with its wording.

| Reference from task files | Source/base location | Result | Finding |
|---|---|---|---|
| `pipeline/`, `airflow/`, `api/`, `web/`, `db/`, `db/migrations/`, `dbt/`, `infra/`, `evals/`, `docs/` | `readme.md` Components | yes | Directory names match. |
| `Makefile`, `.env.example`, `.gitignore`, pre-commit | `readme.md`, `ROADMAP.md` T0.1 | yes | Named by base docs/roadmap. |
| `docker-compose.yml` | `ROADMAP.md` T0.2 | yes | Task is specified. |
| Postgres/pgvector, Neo4j, Redis | `architecture.md`, `ROADMAP.md` T0.2 | yes | Service set matches. |
| `pipeline/config.py`, Pydantic Settings, structured JSON logging | `ROADMAP.md` T0.4, `agents.md` | yes | Exact module path is not prescribed; path is task-plan detail. |
| `core` schema and migration runner | `data_model.md`, `ROADMAP.md` T1.1 | yes | Schema target matches. |
| `NNNN_*.sql` | `agents.md` | yes | Naming is compatible. |
| `sources`, `articles`, `mentions`, `entities`, `events`, `event_participants`, `relations`, `article_chunks`, `entity_embeddings`, `pipeline_runs`, `llm_calls` | `data_model.md` | yes | All named tables are present. |
| HNSW, btree, GIN indexes | `data_model.md` | yes | Index families match. |
| RSS, GDELT, `feedparser`, `trafilatura` | `architecture.md`, `ROADMAP.md` T1.2 | yes | Connector and extraction references match. |
| canonical URL, title/content hash dedup | `architecture.md`, `ROADMAP.md` T1.2 | yes | Near-duplicate wording is equivalent. |
| `data/lake/raw/articles/date=YYYY-MM-DD/*.parquet` | `architecture.md` | yes | Path matches. |
| Parquet, Pandera, DuckDB | `architecture.md`, `data_model.md`, `ROADMAP.md` | yes | Data tools are named in base docs. |
| article/mention/event schemas and schema versions | `data_model.md` | yes | Lake subjects and versioning match. |
| spaCy, transformer model, small CI model, mentions | `architecture.md`, `ROADMAP.md` T1.4 | yes | NER references match. |
| `evals/er/`, 300+ mentions, 60+ articles, JSONL, hard cases | `ENTITY_RESOLUTION.md` | yes | Evaluation references match. |
| Wikidata `wbsearchentities`, top 10, P31/P279, Q5/Q43229/Q6256/Q486972 | `ENTITY_RESOLUTION.md` | yes | Candidate and type references match. |
| `T_high`, `T_low`, optional LLM adjudication | `ENTITY_RESOLUTION.md` | yes | Threshold behavior matches. |
| `pipeline/prompts/<name>.v<N>.md` | `PROMPTS.md` | yes | Prompt path/versioning matches. |
| Pydantic JSON schema, temperature 0, repair retry, prompt hash | `PROMPTS.md`, `agents.md` | yes | Extraction behavior matches. |
| `llm_calls`, token/cost/cache fields | `data_model.md`, `agents.md` | yes | Logging fields match. |
| Neo4j projection, MERGE, rebuild, `make rebuild-graph` | `architecture.md`, `data_model.md`, `ROADMAP.md` | yes | Projection direction matches. |
| `pipeline_runs`, data-quality gates, Pandera row/null checks | `data_model.md`, `architecture.md` | yes | Gate/bookkeeping references match. |
| `python -m pipeline.run --date ... [--stage ...]` | `ROADMAP.md` T1.9 | yes | CLI reference matches. |
| FastAPI, OpenAPI, OTel, Postgres/Neo4j/Redis | `architecture.md`, `ROADMAP.md` T2.1 | yes | API foundation matches. |
| `/entities/search`, `/entities/{id}`, `/entities/{id}/neighbors`, `/graph/path`, `/trending`, `/articles/{id}` | `ROADMAP.md` T2.2 | yes | Endpoint list matches. |
| Redis TTL/cache metrics, rate limiting, pagination | `ROADMAP.md` T2.3/T2.4 | yes | Feature references match. |
| generated OpenAPI client, Next.js 15, force graph, Playwright | `readme.md`, `agents.md`, `ROADMAP.md` | yes | Web references match. |
| Airflow LocalExecutor, `nightly_news_kg`, retries, SLAs, backfill | `ROADMAP.md` T3.1/T3.2 | yes | Orchestration references match. |
| Terraform, AWS, S3, ECR, IAM, SSM/Secrets Manager | `architecture.md`, `ROADMAP.md`, `DECISIONS.md` | yes | IaC references match. |
| ADR-006 through ADR-009 | `DECISIONS.md` | yes | All referenced ADR IDs exist. |
| dbt-duckdb, staging, marts, `analytics`, `/trending` | `data_model.md`, `ROADMAP.md` | yes | Analytics references match. |
| trailing 14-day z-score | `data_model.md` | yes | Formula family matches; edge behavior remains unspecified. |
| HNSW, 1-2 hops, rerank, `/ask`, citations, refusal | `data_model.md`, `PROMPTS.md`, `ROADMAP.md` | yes | GraphRAG references match. |
| recall@k, citation precision, faithfulness, 60+ questions | `ROADMAP.md` T6.4 | yes | Evaluation references match. |
| Grafana Cloud, OTel metrics, k6/Locust, p95, screenshots/write-up | `architecture.md`, `ROADMAP.md` | yes | Hardening references match. |
| `config/models.yaml` | `PROMPTS.md` | yes | Exists as a referenced configuration path, but the file itself is not among base docs. |
| `docs/RUNBOOK.md`, `docs/WRITEUP.md`, `docs/assets/`, `load/` | task plans only | no | Planned future files, not present in base docs. |
| testcontainers, workflow YAML, image vulnerability scanning, chosen graph/client libraries | task plans only | no | Implementation options, not base-doc commitments; unverified. |

## 2. Roadmap/task-section coverage

### Roadmap tasks with no matching task-file section

None. All roadmap IDs T0.1 through T7.4 have a corresponding section in the phase task files.

### Task-file sections with no matching roadmap task

None by task ID. The task plans add implementation detail under existing IDs but do not
introduce a new numbered task.

## 3. Contradictions

- **Article text policy:** `architecture.md` says the raw layer writes fetched article
  text to Parquet, while `agents.md` says article text is processing input only and must
  never be stored in the graph, API, or UI. `data_model.md` says article chunks store
  embeddings and not text. The task plans alternately describe transient text and a
  raw lake containing article data without resolving the storage boundary. **Mismatch.**
- **Lake mutability:** `architecture.md` calls the lake append-only but also requires
  date-partition overwrite on reruns. Task plans repeat atomic replacement. The docs do
  not define whether object versions/partition replacement are considered append-only.
  **Mismatch.**
- **Base path casing:** the requested base files use lowercase `readme.md`,
  `agents.md`, `architecture.md`, and `data_model.md`, while many task references use
  `README.md`, `AGENTS.md`, `ARCHITECTURE.md`, and `DATA_MODEL.md`. On case-sensitive
  systems these are different paths. **Mismatch.**
- **Vector schema:** `data_model.md` leaves vector dimension as configurable `N`, but
  task plans require a migration/index and a configured dimension before ADR-009 is
  resolved. **Mismatch/blocked dependency.**
- **API availability:** task plans require `/trending` endpoint behavior in Phase 2,
  while the roadmap says the trending mart is produced in Phase 5. The plans note this
  dependency but do not define a Phase 2 fallback contract. **Mismatch.**
- **Raw text retention:** task plans include text extraction and transient processing,
  but some acceptance statements say no article body is persisted anywhere, which is
  stronger than the architecture's raw-layer description. **Mismatch.**
- **Local stack scope:** `architecture.md` lists Airflow, API, and web in the local
  compose environment, while Phase 0's acceptance only names Postgres, Redis, and
  Neo4j. **Mismatch in phase boundary, not necessarily behavior.**

## 4. Invented or unverified task-plan content

The following are not fixed by the base docs and are marked **unverified** rather than
being treated as requirements:

- Exact Docker image tags, package versions, supported CI OS, Node version, lockfile,
  and dependency manager.
- Exact migration runner, database driver, testcontainer strategy, and transaction
  implementation.
- Concrete API response shapes, pagination style, cache TTLs, rate limits, cursor
  format, trusted proxy rules, and error payload details.
- Specific OpenAPI client generator, graph visualization library, accessibility target,
  browser matrix, and Next.js rendering strategy.
- Airflow version, timezone, schedule, executor sizing, SLA values, alert destination,
  secret backend, and retry/backoff values.
- AWS region/account layout, Terraform module structure, runtime selection, IAM
  boundaries, S3 lifecycle, ECR retention, OIDC setup, and deployment topology.
- Wikidata response fields, class-closure implementation, User-Agent format, cache TTL,
  API limits, scoring weights, threshold values, and embedding feature behavior.
- LLM provider adapter, model ID, pricing, token accounting, timeout, cost formula,
  and judge model. `PROMPTS.md` explicitly says model IDs/pricing require verification.
- Exact prompt names, initial versions, schemas beyond the listed examples, and release
  enforcement mechanism.
- Parquet schema version policy, S3 atomicity implementation, dbt SQL compatibility,
  source freshness, export strategy, and trending edge-case formula.
- GraphRAG top-k, reranker, context budget, refusal wording, streaming behavior,
  citation validation, and faithfulness threshold.
- Grafana/OTel exporter, metric labels, sampling/retention, alert thresholds, load-test
  tool choice, workload, SLO, and representative dataset size.
- References to testcontainers, security scanning, `make -n`, `git check-ignore`,
  container non-root execution, and artifact retention are **unverified** additions,
  not commitments in the base design docs.

## 5. Acceptance criteria that are not runnable commands

The following task sections contain acceptance criteria that are partly or wholly
observational, manual, or environment-dependent rather than directly runnable commands:

- `phase-0.md`: T0.1 CI passes on an empty scaffold; T0.2 "documented connection probes"
  and intended-volume behavior; T0.3 visible PR checks; T0.4 log-field inspection.
- `phase-1.md`: T1.1 schema introspection; T1.2 no-body persistence policy; T1.4 offset
  and count inspection; T1.5 ER metric/report inspection; T1.7 graph equivalence;
  T1.8 bookkeeping fields; T1.10 no-secrets-in-layers.
- `phase-2.md`: T2.1 readiness/schema behavior; T2.2 stable pagination/index review;
  T2.3 cache metrics and outage behavior; T2.5 mobile usability and DOM/network
  content policy; T2.6 artifact retention and browser flow.
- `phase-3.md`: T3.1 DAG graph/manual run; T3.2 alert/retry outcome; T3.3 CI import behavior.
- `phase-4.md`: T4.1 plan review/no-secret review; T4.2 rollback and secret-source review;
  T4.3 least-privilege and scheduler outcome; T4.4 approval/permissions review;
  T4.5 orphan-resource review.
- `phase-5.md`: T5.1 source freshness documentation; T5.2 grain/formula review;
  T5.3 snapshot atomicity outcome; T5.4 generated-doc inspection.
- `phase-6.md`: T6.1 prohibited-storage inspection; T6.2 context-limit/citation review;
  T6.3 unsupported-claim/refusal behavior; T6.4 human-label quality and three-iteration
  comparison; T6.5 UI/citation behavior.
- `phase-7.md`: T7.1 dashboard/runbook/tabletop outcomes; T7.2 report interpretation;
  T7.3 clean-user README walkthrough and screenshot review; T7.4 write-up/reviewer checklist.

These criteria can be made demonstrable with scripts or tests later, but the task files
currently mix commands, assertions, and human review. No fixes were made as requested.
