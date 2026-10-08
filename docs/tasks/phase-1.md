# Phase 1: Core Pipeline

## Exit criteria
A one-day run completes locally in Docker, ER F1 is reported, and pipeline test coverage is at least 80%.

## T1.1 Canonical schema and migrations
- **Goal:** Implement the `core` Postgres schema in `docs/DATA_MODEL.md` with forward-only migrations and a migration runner.
- **Files:** Create `db/migrations/NNNN_*.sql`, migration tooling, schema tests, and Make targets.
- **Approach:** Add tables, enums/checks, foreign keys, uniqueness, indexes, fixed 384-dimensional versioned embedding tables, and timestamps; validate `EMBEDDING_DIM` at startup against the table version and fail loudly on mismatch (ADR-013); make reruns safe through the migration tool, not ad hoc SQL.
- **Dependencies:** Phase 0; ADR-009 fixes the initial 384-dimensional versioned embedding tables and requires `EMBEDDING_MODEL`/`EMBEDDING_DIM`; ADR-013 defines startup validation.
- **Acceptance:** `make migrate` applies all migrations to an empty Postgres; second run is a no-op; introspection shows required tables/constraints/indexes; rollback is not assumed because migrations are forward-only.
- **Tests:** Fresh database integration tests, constraint/upsert tests, and migration idempotency.
- **Risks/unknowns:** Enum evolution, migration runner choice, and exact article status transition rules are unspecified. Embedding dimension is fixed per table version (ADR-009/013).
- **Do not:** Create Neo4j as a second source of truth or store article text in graph/API records.

## T1.2 Ingestion connectors
- **Goal:** Ingest RSS and GDELT metadata/text processing inputs with canonical URLs, deduplication, extraction, and rate limiting.
- **Files:** Create connector modules, canonicalization/dedup utilities, persistence adapters, fixtures, and tests.
- **Approach:** Define typed connector records; normalize URLs; deduplicate by URL and title/content hash; fetch/extract text only as processing input and persist permitted text only under private `lake/private/article_text/` with 30-day expiry; respect robots.txt/source terms and use feed summaries when needed; persist metadata and hashes; handle retries and per-source rate limits; make date reruns upsert-safe.
- **Dependencies:** T1.1 and config/logging; source credentials and GDELT API behavior need verification.
- **Acceptance:** Fixture RSS/GDELT inputs produce deterministic article metadata; duplicate URL/hash yields one article; failed fetch is represented with status/error metadata; permitted text is private, expires after 30 days, and is absent from Postgres/API/logs/fixtures; repeated date run has unchanged row counts.
- **Tests:** Parser fixtures, URL normalization, dedup, retry/rate-limit, extraction failure, and idempotency tests; mocked external calls only.
- **Risks/unknowns:** GDELT endpoint/schema, publisher robots/ToS, canonical URL rules, and retention of transient text are not fully specified.
- **Do not:** Scrape indiscriminately, bypass publisher restrictions, or expose/store text outside private `lake/private/article_text/`.

## T1.3 Lake writer and schemas
- **Goal:** Write versioned immutable Parquet run outputs with an atomic active-manifest update and Pandera validation.
- **Files:** Create lake writer, schemas, fixture data, partition tests, and config docs.
- **Approach:** Define article/mention/event schemas and schema versions; write immutable outputs to `lake/<layer>/<table>/date=D/run_id=R/*.parquet`, validate null rates/types/row counts, then write `lake/_manifests/<table>/date=D.json` as the active-run pointer last. Cleanup superseded outputs per `RUN_RETENTION_DAYS` (default 14) and never delete a run referenced by any current manifest (ADR-010/012); retain private article text for exactly 30 days (ADR-002). Document that backfills older than 30 days cannot rerun steps requiring article text.
- **Dependencies:** T1.2; event/mention fields from later tasks must be versioned rather than guessed.
- **Acceptance:** `make ingest-sample` writes an immutable run under `lake/<layer>/<table>/date=D/run_id=R/*.parquet` and writes the active manifest last; invalid rows fail before pointer update; old runs are not selected; DuckDB resolves the fixture through the manifest.
- **Tests:** Schema validation, failed-run manifest stability, manifest switch, retention protection for referenced runs, empty input, and round-trip tests.
- **Risks/unknowns:** S3 manifest-write consistency and exact schema version policy are unspecified; cleanup must remain manifest-safe.
- **Do not:** Treat run outputs as mutable row storage, bypass the manifest, or write unvalidated partitions.

## T1.4 NER
- **Goal:** Extract typed mentions with configurable spaCy models and persist auditable offsets.
- **Files:** Create NER stage, model configuration, mention repository, fixtures, and tests.
- **Approach:** Process permitted text from private `lake/private/article_text/`; use configurable model (small CI model); emit surface, label, offsets, sentence hash, and article ID; validate offsets and supported labels; update article status transactionally.
- **Dependencies:** T1.1 and T1.2; model availability must be verified.
- **Acceptance:** Fixture text yields deterministic mentions with valid non-overlapping offsets; unsupported labels are handled by documented mapping; rerun does not duplicate mentions; counts are logged.
- **Tests:** Golden NER fixtures, offsets, empty text, model failure, transaction/idempotency tests.
- **Risks/unknowns:** Transformer model size/performance, label mapping, and sentence hash algorithm are unspecified.
- **Do not:** Persist article text outside private `lake/private/article_text/`; never put it in Postgres, graph, logs, fixtures, or API responses.

## T1.5 Entity resolution and evaluation
- **Goal:** Link mentions to Wikidata QIDs or clustered NIL entities per the resolution design and report metrics.
- **Files:** Create candidate client/cache, normalizer/scorer, resolver, repositories, `evals/er/` runner/data format, and tests.
- **Approach:** Normalize; query/cache top candidates with User-Agent and rate limits; type-filter; score documented features; apply configurable thresholds; persist score/method on every mention; cluster NILs; produce precision/recall/F1 and per-type metrics.
- **Dependencies:** T1.4; human-labeled 300+ mention set is a manual dependency; Wikidata APIs and popularity fields require verification.
- **Acceptance:** Offline fixture run produces deterministic decisions; cache prevents duplicate requests; every mention has score/method; evaluator emits link/NIL/per-type metrics; before/after report is possible for changes.
- **Tests:** Normalization, cache TTL, type filtering, thresholds, NIL clustering, API failure/rate limits, and evaluator tests.
- **Risks/unknowns:** Wikidata response/rate limits, class closure implementation, threshold values, and embedding feature availability.
- **Do not:** Invent labels, silently resolve uncertain cases, or remove hard cases from evaluation data.

## T1.6 LLM extraction
- **Goal:** Extract schema-valid events/relations with caching, retries, and usage logging.
- **Files:** Create versioned prompts, Pydantic output schemas, provider adapter, cache, persistence, and tests.
- **Approach:** Load model/temperature/token settings from config; hash prompt version/model/input; send only allowed metadata, text transiently, and resolved IDs; validate JSON; perform one repair retry; mark failures; record token/cost/cache fields.
- **Dependencies:** T1.4/T1.5, T1.1, and ADR-009's configured local 384-dimensional embedding baseline; API-model pricing is only needed for the later bake-off.
- **Acceptance:** Fixture provider returns events/relations constrained to supplied IDs; malformed output gets one repair then failed status; cache hit avoids a call; `llm_calls` records usage; summaries obey length rule.
- **Tests:** Schema validation, prompt hashing, cache, retry, refusal of unknown IDs, cost logging, and failure status tests with mocked provider.
- **Risks/unknowns:** Provider API, model IDs/pricing, token counting, and cost calculation are explicitly unverified.
- **Do not:** Hard-code model IDs, accept unvalidated JSON, or include outside knowledge in extraction.

## T1.7 Neo4j projection
- **Goal:** Build an idempotent Neo4j projection from Postgres and support rebuild.
- **Files:** Create projector, Cypher constraints/queries, CLI/Make target, and integration tests.
- **Approach:** Read canonical rows; use stable IDs and MERGE; keep Postgres full history while projecting Entity/Event nodes, weighted entity edges, and Article nodes only within `ARTICLE_WINDOW_DAYS` (default 14); emit `CO_OCCURS_WITH` only above `CO_OCCUR_MIN_WEIGHT` for top-K entities/day and prune by `last_seen`. Check Neo4j counts before writes; at 90% of `NEO4J_MAX_NODES`/`NEO4J_MAX_RELS`, fail soft by skipping lowest-priority edges and logging skipped count. Use bounded retry for a paused instance to resume. Isolate projection transactions; implement drop/rebuild from Postgres; never write graph-only facts.
- **Dependencies:** T1.1 and extracted data; Neo4j local service; ADR-008 defines sizing and projection policy; actual Aura limits must be verified in the console (TODO(verify)).
- **Acceptance:** Projecting twice produces same node/relationship counts; changed canonical data updates projection; `make rebuild-graph` recreates equivalent bounded graph; constraints exist; article nodes contain metadata only and stay within the rolling window; a synthetic near-cap fixture confirms lowest-priority edges are skipped and counted; paused-instance retry is bounded.
- **Tests:** Testcontainer/local integration, idempotency, rebuild equivalence, constraint, and failure/retry tests.
- **Risks/unknowns:** Neo4j driver/version and relationship aggregation semantics remain unspecified; actual Aura limits require console verification (TODO(verify)).
- **Do not:** Dual-write from pipeline stages or store article text.

## T1.8 Quality gates and run bookkeeping
- **Goal:** Enforce stage data quality and record pipeline run counts/durations/statuses.
- **Files:** Create gate utilities, `pipeline_runs` repository, stage wrappers, and tests.
- **Approach:** Define per-stage schema/null/row-count checks; fail before downstream loads; record start/end/status/error metadata in finally-safe code; use structured logs.
- **Dependencies:** T1.1-T1.7.
- **Acceptance:** Invalid fixture blocks downstream stage; successful run records rows in/out and duration; failed run records failure and reason; rerun remains backfill-safe.
- **Tests:** Gate failure, zero-row policy, bookkeeping on exception, and transaction boundary tests.
- **Risks/unknowns:** Threshold values and whether zero rows are valid per source/day are unspecified.
- **Do not:** Log raw article text, swallow gate failures, or make checks advisory.

## T1.9 CLI
- **Goal:** Expose date/stage-selectable pipeline entrypoints shared by Airflow later.
- **Files:** Create `pipeline/run.py`, command parsing, stage registry, and CLI tests.
- **Approach:** Validate ISO date; select ordered stages; pass typed config/context; make each stage idempotent; return nonzero on failure and emit counts.
- **Dependencies:** T1.2-T1.8.
- **Acceptance:** `python -m pipeline.run --date 2026-01-01 --stage ingest` runs only ingest; full date run executes ordered stages; invalid date/stage exits nonzero with usage; repeated run is safe.
- **Tests:** Argument parsing, stage selection/order, failure exit code, and dry fixture integration.
- **Risks/unknowns:** Exact stage names and partial rerun semantics are not specified.
- **Do not:** Put orchestration logic in future DAGs or add hidden global state.

## T1.10 Pipeline images
- **Goal:** Build reproducible pipeline Docker image(s) and verify in CI.
- **Files:** Create pipeline Dockerfile, ignore file, compose service additions if needed, and CI build step.
- **Approach:** Multi-stage or minimal image; install locked dependencies; run non-root; inject config at runtime; execute CLI entrypoint; use sample fixture for smoke test.
- **Dependencies:** T1.9 and Phase 0 compose.
- **Acceptance:** `docker build` succeeds; container runs sample date command with mounted/configured dependencies; CI builds image; no secrets are in layers.
- **Tests:** Image build and smoke test; dependency vulnerability scan if adopted and verified.
- **Risks/unknowns:** Model download size, system packages, and registry strategy are unspecified.
- **Do not:** Bake credentials, production endpoints, or article fixtures containing prohibited text into the image.
