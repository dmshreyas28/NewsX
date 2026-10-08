# Decision log (ADRs)
Format: ID | Status | Context | Decision | Alternatives rejected | Consequences.
The human owner rewrites the "why" in their own words before a phase closes.

- ADR-001 Accepted: Postgres is system of record; Neo4j is a rebuildable projection.
  Rejected: dual-writes (drift risk). Consequence: extra projector step.
- ADR-002 Accepted (amended): Fetched article text is processing input and may be
  persisted only under private `lake/private/article_text/`, with no public access and
  30-day lifecycle expiry, to support reproducible reruns, backfills, and evals. It is
  never stored in Postgres or Neo4j, returned by the API, rendered in the UI, logged,
  or committed in fixtures. Robots.txt and source terms are respected; feed summaries
  are used where full text is not permitted. Rejected: no persistence (not reproducible)
  and serving-layer storage (copyright/ToS risk). Consequence: private storage,
  cleanup, access controls, and synthetic/license-clear fixtures are required.
- ADR-003 Accepted: Airflow over Dagster/Prefect for market familiarity.
  Consequence: heavier to run; see ADR-007.
- ADR-004 Accepted: dbt + DuckDB over cloud warehouse to avoid multi-cloud and cost.
- ADR-005 Accepted: Terraform on AWS; S3 lake.
- ADR-006 OPEN: API runtime on AWS (Lambda via container vs App Runner vs Fargate).
  Criteria: cost at idle, cold-start, pgvector/Neo4j connection handling. Verify current pricing.
- ADR-007 OPEN: where the scheduled production pipeline runs (single small EC2 running
  compose vs ECS scheduled task vs local-only Airflow with cloud run via GitHub Actions).
  Airflow needs an always-on host; cost vs "$0" goal must be decided explicitly.
- ADR-008 Accepted: Design Neo4j for the AuraDB Free lower bound of 50,000 nodes and
  175,000 relationships (TODO(verify) actual limits in the Aura console when creating
  the instance; configure `NEO4J_MAX_NODES` and `NEO4J_MAX_RELS`). Postgres keeps full
  history. Neo4j holds Entity and Event nodes, weighted entity-entity edges, and Article
  nodes only within a rolling window (`ARTICLE_WINDOW_DAYS`, default 14). Write
  `CO_OCCURS_WITH` edges only above `CO_OCCUR_MIN_WEIGHT` and for top-K entities per day;
  prune by `last_seen`. The projector checks counts before writes and fails soft at 90%
  of caps by skipping lowest-priority edges and logging the skipped count. It tolerates
  paused instances with bounded retry for resume. Rejected: projecting all history and
  unbounded co-occurrence edges. Consequence: graph is a bounded projection; Postgres
  remains the complete history.
- ADR-009 Accepted: Start with a local open-source sentence-embedding model in the
  384-dimension class, default `all-MiniLM-L6-v2`; verify its model card and license
  (TODO(verify)). Configure `EMBEDDING_MODEL` and `EMBEDDING_DIM`. Store
  `embedding_model` on every embedding row and use versioned tables
  `article_chunks_v1` and `entity_embeddings_v1`. A model change creates a new
  versioned table and reindex job, never an in-place ALTER. Rejected: in-place dimension
  changes and API-only embeddings as the initial default. Consequence: local inference
  is the baseline and T6.1 must run an eval bake-off against an API model.
- ADR-010 Accepted: Lake outputs use immutable run paths
  `lake/<layer>/<table>/date=D/run_id=R/*.parquet` and a mutable active pointer at
  `lake/_manifests/<table>/date=D.json`, written last. Readers, including dbt, resolve
  active runs through manifests; configured lifecycle cleanup removes old runs.
  Rejected: in-place partition replacement and readers scanning all runs. Consequence:
  manifest validation and cleanup are required.
- ADR-011 Accepted: Define the `/trending` OpenAPI contract in Phase 2. Initially query
  Postgres mentions using 24-hour count versus trailing 14-day mean/std z-score; in
  Phase 5 replace only the implementation with `analytics.mart_trending_entities`.
  Rejected: delaying the endpoint contract until dbt exists. Consequence: one contract
  test must pass against both implementations.
- ADR-012 Accepted: `RUN_RETENTION_DAYS` (default 14) governs superseded lake run
  outputs; private article text expires at a fixed 30 days per ADR-002. Cleanup must
  never delete a run referenced by any current manifest. Backfills older than 30 days
  cannot rerun steps requiring article text. Rejected: deleting active runs or making
  private-text expiry dependent on run retention. Consequence: cleanup checks manifests
  and run age, and backfill procedures document the text-dependent limitation.
- ADR-013 Accepted: Embedding dimension is hard-coded per versioned table.
  `EMBEDDING_DIM` is validated at startup against the table version and fails loudly on
  mismatch. Rejected: silently using a dimension that differs from the table schema.
  Consequence: each table version has a fixed dimension and config validation prevents
  incompatible writes.
