# Decision log (ADRs)
Format: ID | Status | Context | Decision | Alternatives rejected | Consequences.
The human owner rewrites the "why" in their own words before a phase closes.

- ADR-001 Accepted: Postgres is system of record; Neo4j is a rebuildable projection.
  Rejected: dual-writes (drift risk). Consequence: extra projector step.
- ADR-002 Accepted: Store derived data only, not article text (copyright/ToS risk).
- ADR-003 Accepted: Airflow over Dagster/Prefect for market familiarity.
  Consequence: heavier to run; see ADR-007.
- ADR-004 Accepted: dbt + DuckDB over cloud warehouse to avoid multi-cloud and cost.
- ADR-005 Accepted: Terraform on AWS; S3 lake.
- ADR-006 OPEN: API runtime on AWS (Lambda via container vs App Runner vs Fargate).
  Criteria: cost at idle, cold-start, pgvector/Neo4j connection handling. Verify current pricing.
- ADR-007 OPEN: where the scheduled production pipeline runs (single small EC2 running
  compose vs ECS scheduled task vs local-only Airflow with cloud run via GitHub Actions).
  Airflow needs an always-on host; cost vs "$0" goal must be decided explicitly.
- ADR-008 OPEN: Neo4j retention policy sized to Aura Free limits (verify limits).
- ADR-009 OPEN: embedding model and dimension (affects schema, cost, eval results).
