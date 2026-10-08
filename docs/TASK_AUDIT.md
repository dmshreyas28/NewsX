# Task Audit

This is the re-audit of `docs/tasks/phase-0.md` through `phase-7.md` after the
filename and architecture decisions in `docs/DECISIONS.md`. No task plan was
changed solely to hide an audit finding.

## Resolved findings

- **ADR-002 amended:** The task plans now allow fetched text only under private
  `lake/private/article_text/` with 30-day expiry, and prohibit Postgres, Neo4j,
  API, UI, logs, and committed fixtures. Tasks require robots/terms compliance and
  feed summaries where full text is unavailable. **Resolved by ADR-002.**
- **Lake replacement contradiction:** Task T1.3 now uses immutable
  `lake/<layer>/<table>/date=D/run_id=R/*.parquet` outputs and a last-written
  `lake/_manifests/<table>/date=D.json` active pointer. Readers resolve manifests;
  cleanup is lifecycle/config driven. **Resolved by ADR-010.**
- **Embedding dimension/versioning:** T1.1 and T6.1 use the 384-dimensional baseline,
  `EMBEDDING_MODEL`, `EMBEDDING_DIM`, `embedding_model`, and versioned tables.
  Startup validation now rejects a mismatch with the table version. **Resolved by
  ADR-009 and ADR-013; model card/license remains TODO(verify).**
- **Neo4j capacity and projection policy:** The projection is bounded by configured
  node/relationship caps, rolling Article window, edge selection/pruning, soft-cap
  behavior, and paused-instance retry. **Resolved by ADR-008; actual Aura console limits
  remain TODO(verify).**
- **Lake/private-text retention:** Superseded lake runs use `RUN_RETENTION_DAYS=14`
  default with manifest-safe deletion; private article text expires at 30 days and
  older text-dependent backfills cannot rerun. **Resolved by ADR-012 and ADR-002.**
- **`/trending` sequencing:** Phase 2 owns the OpenAPI contract and a Postgres
  24-hour versus trailing-14-day z-score implementation. Phase 5 swaps only the
  implementation to the analytics mart and must run the same contract test.
  **Resolved by ADR-011.**
- **Filename case:** Base references are now intended to use `README.md`,
  `AGENTS.md`, `docs/ARCHITECTURE.md`, and `docs/DATA_MODEL.md`. **Resolved by the
  case-only git moves in this change.**

## 1. Reference audit

| Reference category | Result | Notes |
|---|---|---|
| Base filenames and cross-links | yes | Uppercase names are tracked and task references use uppercase names. |
| Pipeline, API, web, Airflow, dbt, Terraform, evals directories | yes | Match `README.md` and `docs/ROADMAP.md`. |
| Core tables, indexes, graph nodes/relationships | yes | Match `docs/DATA_MODEL.md`; embedding tables are now versioned. |
| RSS/GDELT/feedparser/trafilatura, spaCy, Wikidata | yes | Match `docs/ARCHITECTURE.md` and `docs/ENTITY_RESOLUTION.md`. |
| Prompt versions, JSON schema, temperature 0, retry/cache/logging | yes | Match `docs/PROMPTS.md` and `AGENTS.md`. |
| ADR-001 through ADR-013 | yes | ADR-006 and ADR-007 remain open; ADR-008, ADR-009, ADR-010, ADR-011, ADR-012, and ADR-013 are accepted. |
| `docs/API.md` contract | yes | Roadmap and Phase 2 task plan both define T2.0, and T2.1-T2.6 depend on it. |
| `config/models.yaml` | yes | Referenced by `PROMPTS.md`, but the file is not created by the base-doc task. |
| `docs/RUNBOOK.md`, `docs/WRITEUP.md`, `docs/assets/`, `load/` | no | Planned outputs only; not base documents. |

## 2. Roadmap coverage

Every roadmap task ID has a matching section in `docs/tasks/`, and every task-file
section ID has a matching task in `docs/ROADMAP.md`. This includes Phase 2 T2.0.
The dependency note in the roadmap and task plan specifies that T2.1-T2.6 depend on T2.0.

## 3. Remaining contradictions or issues

- The base architecture says fetched text is processing input but permits private
  lake persistence, while some task wording still says "transient" or "no article
  body" without explicitly naming the private path. These phrases are not intended
  to authorize another storage location; standardize them during implementation.
- `docs/DATA_MODEL.md` uses `vector(384)`; the actual migration/index syntax remains
  an implementation detail, while ADR-013 now requires startup mismatch validation.
- ADR-008's 50,000-node/175,000-relationship AuraDB Free values are lower-bound design
  assumptions; actual limits must be verified in the Aura console before deployment.
- `RUN_RETENTION_DAYS` defaults to 14 while private article text has fixed 30-day expiry;
  actual object lifecycle/cleanup behavior still needs implementation and verification.
- The base `README.md` quickstart still references `data/lake` generically and does
  not explain manifest resolution or private article-text restrictions.
- T2.2's exact non-trending response shapes and graph-path semantics remain undefined.
- The API error schema, pagination format, cache TTLs, rate limits, trusted proxy
  behavior, and endpoint limits remain undefined.
- ADR-006 and ADR-007 remain open; API runtime and production scheduler decisions are
  not settled.
- `PROMPTS.md` still requires verification of model IDs/pricing; ADR-009 requires
  verification of the `all-MiniLM-L6-v2` model card and license.
- Wikidata response fields, rate limits, User-Agent format, scoring weights, and ER
  thresholds remain undefined.
- dbt source freshness, SQL compatibility, export atomicity, and trending behavior
  for missing/zero-variance baselines remain undefined.
- GraphRAG top-k, reranking, context budget, refusal schema, streaming behavior,
  and faithfulness evaluation thresholds remain undefined.
- OTel/Grafana exporter, metric labels, retention, alert thresholds, load profile,
  SLO, and representative data size remain undefined.
- `docs/API.md` has not yet been written; Phase 2 T2.0 requires it before the other
  Phase 2 tasks.
- The remaining Phase 2 API decisions needed for `docs/API.md` include exact
  non-trending response schemas, pagination, error payload, graph path semantics, and
  endpoint bounds.

## 4. Unverified or task-plan-only additions

The following remain **unverified** and are not claims in the base docs: Docker image
tags; dependency/tool versions; CI OS and lockfile strategy; migration runner; exact
OpenAPI client and graph libraries; testcontainer use; Airflow version/timezone/SLA;
Terraform module layout and AWS runtime details; Wikidata scoring implementation;
provider adapters, token accounting, and pricing; dbt export mechanism; reranker and
judge model; OTel exporter; security scanner; and k6/Locust choice.

The default `all-MiniLM-L6-v2` name and license are also **TODO(verify)** under ADR-009.

## 5. Acceptance criteria not directly runnable

The following remain partly manual or observational: CI visibility and PR behavior;
connection/health and volume inspection; schema/index inspection; ER metric review;
Neo4j rebuild equivalence; secret/image-layer review; API readiness and mobile UI
behavior; Airflow alerts and retry outcomes; Terraform plan/rollback/least-privilege
review; dbt docs and snapshot review; embedding bake-off review; GraphRAG refusal and
citation quality; dashboard/runbook tabletop exercise; load-test report interpretation;
README walkthrough/screenshots; and write-up review.

These are reported, not fixed, because this audit task explicitly prohibits unrelated
implementation changes.
