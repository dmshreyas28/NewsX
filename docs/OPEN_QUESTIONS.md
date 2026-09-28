# Open Questions

This file records ambiguity, contradiction, and unverified facts found in the supplied README, AGENTS.md, and design documents. Resolve an item before the dependent task is considered complete; record consequential decisions in `docs/DECISIONS.md`.

## Decisions explicitly left open

### ADR-006: API runtime
- Choose Lambda container, App Runner, or Fargate.
- Compare current idle cost, cold start, request/runtime limits, connection pooling, Neo4j/Postgres networking, observability, and operational burden.
- Required before Phase 4 T4.2.

### ADR-007: Production scheduler
- Choose a small EC2 compose host, ECS scheduled task, or local-only Airflow invoking cloud runs via GitHub Actions.
- Compare always-on cost, Airflow persistence, retries/backfills, secrets, networking, and failure recovery.
- Required before Phase 4 T4.3.

### ADR-008: Neo4j retention
- Verify current Aura Free node/relationship/storage/inactivity limits.
- Select retention/pruning and aggregation policy that preserves useful graph behavior.
- Required before production projection; limits are explicitly marked TODO(verify) in the data model.

### Resolved decisions
- ADR-002 amended the article-text storage policy; see `docs/DECISIONS.md`.
- ADR-009 resolved the initial 384-dimensional embedding baseline and versioned-table
  migration policy; verify the model card and license before implementation.
- ADR-010 resolved immutable run outputs plus manifest pointers for lake reads.
- ADR-011 resolved the `/trending` contract and Phase 2/Phase 5 implementation split.

## Cross-document ambiguities

- The repository is described as pre-implementation, but no source files were present when this plan was generated; confirm whether the pasted documents are the intended baseline.
- README says `make migrate && make ingest-sample`, while the roadmap also requires `make check`, `make test`, `make api`, `make web`, `make rebuild-graph`, and `make ingest-sample`; define the complete Make target contract and expected prerequisites.
- AGENTS requires mypy strict on new Python code and TypeScript strict/ESLint/Prettier, but supported Python/Node versions, package managers, lockfile formats, and exact tool versions are not specified.
- AGENTS says every task ends with tests, lint/type checks, and docs updates; some roadmap tasks are infrastructure/manual tasks where a local check may be impossible. Define the CI substitute and evidence standard.
- ADR-010 resolves the prior append-only/overwrite wording using immutable run outputs
  and a manifest pointer; define the cleanup value N during implementation.
- ADR-002 resolves private article-text persistence, access, and 30-day expiry; the
  remaining implementation question is the exact storage adapter and lifecycle test.
- ADR-009 resolves the initial embedding dimension and versioned tables; verify the
  model card/license and define migration SQL during T1.1.
- Event extraction says input may include truncated text, while the article text policy is ambiguous for transient processing and logs. Define maximum token budget and redaction/logging guarantees.
- Article status values are listed, but allowed transitions, retry/failed reason storage, and whether stages may skip statuses are undefined.
- “Merge same-surface mentions per article” conflicts with mention offsets and potentially distinct contexts; define merge key and whether all spans are retained.
- NER label mapping lists `PERSON|ORG|GPE|...` but does not define the full enum or handling for spaCy labels outside it.
- Entity resolution requires Wikidata P31/P279 closure and sitelink popularity, but the retrieval method, closure cache, limits, and response fields are unspecified.
- Entity resolution says optional LLM adjudication, while the LLM spec says extraction only; define provider/config/schema/cost/evaluation behavior for adjudication.
- Wikidata endpoint, User-Agent format, rate limits, cache TTL, licensing, and availability are unverified.
- Candidate cache storage/schema is not defined in DATA_MODEL.
- “Thresholds in config” has no initial values, calibration procedure, or versioning requirement.
- The mandatory 300+ mentions across 60+ articles and 60+ RAG questions are human-labeling tasks; define dataset ownership, privacy/licensing, annotation format, and completion gate.
- LLM model names/pricing are explicitly unverified; provider API, token accounting, retry limits, timeout, and cost formula are not defined.
- Prompt version files are required, but no initial prompt names/version IDs or release/immutability mechanism is specified.
- `llm_calls.cost_usd` requires pricing, currency, and cached-call accounting rules; define behavior when provider usage is unavailable.
- Neo4j relationship aggregation (`MENTIONED_IN.count`, co-occurrence weight, `last_seen`) and refresh semantics are not defined.
- API response shapes, pagination style, graph path semantics, error schema, limits, and trending behavior before dbt Phase 5 are unspecified.
- API says Redis cache TTLs but no TTL values, invalidation triggers, serialization/versioning, or behavior during Redis outage are defined.
- Auth-free rate limiting has no algorithm, limits, trusted proxy policy, or deployment assumptions.
- OpenAPI client generator and version are unspecified; generated output location and CI drift check are not defined.
- Next.js UI design, graph library, browser support, accessibility target, and whether server/client rendering is required are unspecified.
- Airflow version, timezone/schedule, connection/secret strategy, SLA values, alert destination, and catchup default are unspecified.
- Terraform state bootstrap, AWS region/account, environment model, S3 lifecycle, ECR retention, and IAM boundaries are unspecified.
- AWS managed services are external in manual setup, but infrastructure ownership and network allowlists for Neon, Aura, Redis, and Grafana are not defined.
- Current AWS pricing, free-tier eligibility, and Grafana/Upstash/Neo4j limits are unverified and must not be claimed.
- dbt source path, schema-version handling, freshness policy, SQL compatibility, and export mechanism to Postgres are unspecified.
- Trending z-score needs timezone, baseline availability, zero standard deviation, missing days, and ranking/tie rules.
- GraphRAG top-k, graph expansion depth/filters, reranking method, context budget, answer schema, refusal wording, and streaming behavior are unspecified.
- Citation rules say citations map to article URLs, but the article ID/URL mapping, duplicate URLs, and invalid citation behavior need definition.
- Faithfulness evaluation and LLM judge model/thresholds are not defined; judge results must be separated from human spot checks.
- Observability exporter, metric names/labels, sampling, retention, dashboard ownership, and alert thresholds are unspecified.
- Load-test tool, traffic model, dataset size, SLO, cache warmness, and acceptable p95 are unspecified.

## Verification policy

Do not fill these gaps with guessed provider limits, model IDs, package versions, pricing, or API behavior. When implementation requires a choice, verify it from the authoritative provider documentation, record the date/source and decision in `DECISIONS.md`, and add a test or configuration validation for the chosen contract.
