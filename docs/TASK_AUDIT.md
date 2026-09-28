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
  **Resolved by ADR-009; model card/license remains TODO(verify).**
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
| ADR-001 through ADR-011 | yes | ADR-009, ADR-010, and ADR-011 are now present; ADR-006 through ADR-008 remain open. |
| `config/models.yaml` | yes | Referenced by `PROMPTS.md`, but the file is not created by the base-doc task. |
| `docs/RUNBOOK.md`, `docs/WRITEUP.md`, `docs/assets/`, `load/` | no | Planned outputs only; not base documents. |

## 2. Roadmap coverage

Every roadmap task T0.1 through T7.4 has exactly one matching section in the phase
task files. No task-file section has an ID absent from `docs/ROADMAP.md`.

## 3. Remaining contradictions or issues

- The base architecture says fetched text is processing input but permits private
  lake persistence, while some task wording still says "transient" or "no article
  body" without explicitly naming the private path. These phrases are not intended
  to authorize another storage location; standardize them during implementation.
- `docs/DATA_MODEL.md` uses `vector(384)` but the actual migration/index syntax and
  whether `EMBEDDING_DIM` may differ by environment are not specified.
- `docs/ARCHITECTURE.md` says cleanup occurs after N days, while ADR-002 fixes the
  private article-text expiry at 30 days. Define whether run-output cleanup N may be
  different from the private-text expiry.
- The base `README.md` quickstart still references `data/lake` generically and does
  not explain manifest resolution or private article-text restrictions.
- T2.2's exact non-trending response shapes and graph-path semantics remain undefined.
- The API error schema, pagination format, cache TTLs, rate limits, trusted proxy
  behavior, and endpoint limits remain undefined.
- ADR-006, ADR-007, and ADR-008 remain open; runtime, scheduler, and Neo4j retention
  cannot be treated as settled deployment requirements.
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
