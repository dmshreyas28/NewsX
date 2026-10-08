# Phase 0: Foundations

## Exit criteria
`make check` is green in CI on an empty-but-wired repository and `docker compose up` brings the local services to healthy state.

## T0.1 Repo scaffold
- **Goal:** Establish the documented directory layout, developer commands, dependency/tool configuration, and safe environment defaults.
- **Files:** Create `pipeline/`, `airflow/`, `api/`, `web/`, `db/`, `db/migrations/`, `dbt/`, `infra/`, `evals/`, `docs/`, `Makefile`, `.env.example`, `.gitignore`, pre-commit config, package manifests, and minimal package placeholders. Modify `README.md` only if commands differ from the documented contract.
- **Approach:** Create directories; choose and document Python/Node dependency workflows; make targets delegate to tools without embedding business logic; exclude secrets, local data, caches, and generated artifacts; add trivial import/build placeholders so checks can run.
- **Dependencies:** None. Must precede all later phases.
- **Acceptance:** `make check` exits 0 locally; `make -n check` shows lint, type, test, and frontend checks; `git check-ignore .env data` reports ignored paths; `pre-commit run --all-files` runs cleanly. CI acceptance belongs to T0.3 only.
- **Tests:** Local smoke tests for package imports and Make targets; pre-commit hooks on all files.
- **Risks/unknowns:** Exact dependency versions and whether all tools are available on CI are unspecified. Keep versions/configuration explicit and record any unverified choice.
- **Do not:** Implement ingestion, schemas, endpoints, or placeholder behavior that pretends to process data.

## T0.2 Local services
- **Goal:** Run Postgres with pgvector, Neo4j, and Redis locally with deterministic health checks.
- **Files:** Create `docker-compose.yml`, service-specific config directories if needed, and setup documentation updates.
- **Approach:** Pin image tags only after verification; define named volumes, required env vars, network wiring, ports, healthchecks, and least-privilege local credentials; ensure restart and startup ordering use health conditions where supported.
- **Dependencies:** T0.1; manual credentials from `docs/MANUAL_SETUP.md` are not required for local defaults.
- **Acceptance:** `docker compose config` exits 0; `docker compose up -d postgres redis neo4j` starts all three; `docker compose ps` reports healthy; documented connection probes succeed; `docker compose down` preserves only intended named volumes.
- **Tests:** Documented local Compose config validation and service health smoke test; `make check` and `pre-commit run --all-files` pass locally. CI acceptance belongs to T0.3 only.
- **Risks/unknowns:** Neo4j and pgvector image tags, authentication defaults, resource requirements, and health endpoint behavior require verification.
- **Do not:** Add Airflow, API, web, or production cloud deployment in this task.

## T0.3 CI
- **Goal:** Run lint, type checks, and tests on pull requests with dependency caching.
- **Files:** Create `.github/workflows/ci.yml` and cache/tool config; modify `Makefile` only as needed.
- **Approach:** Matrix only where justified; install supported Python and Node versions; cache uv/pip and npm data using lockfile keys; run `make check`; fail on missing generated clients only after their generation task exists.
- **Dependencies:** T0.1 and T0.2.
- **Acceptance:** A pull request workflow triggers on PRs; each required check is visible; a deliberately failing lint/test causes a failed workflow; clean scaffold passes. The first CI run passes on the combined output of T0.1 and T0.2 (neither was run in CI). Enable branch protection requiring the CI check only after this passes (human task; see `docs/MANUAL_SETUP.md`).
- **Tests:** Validate workflow YAML and run the commands locally in the CI order.
- **Risks/unknowns:** No supported CI OS/version or lockfile strategy is specified.
- **Do not:** Add deployment, credentials, scheduled jobs, or ignored failures.

## T0.4 Configuration and logging
- **Goal:** Provide Pydantic Settings configuration and structured JSON logging shared by pipeline and API.
- **Files:** Create `pipeline/config.py` or shared config package, logging utility, tests, and `.env.example` entries.
- **Approach:** Define typed environment-backed settings; fail clearly on invalid required values; separate local/test defaults from production secrets; emit JSON with task, run date, counts, duration, and correlation fields where available.
- **Dependencies:** T0.1; service variable names from T0.2.
- **Acceptance:** A test settings instance loads without secrets; invalid values fail validation; a sample log parses as JSON and contains stable fields; `.env.example` contains names but no credentials; `make check` remains green.
- **Tests:** Settings validation, precedence, redaction, and logging serialization tests.
- **Risks/unknowns:** Exact environment variable names, logger backend, OpenTelemetry integration timing, and cost fields are not fixed.
- **Do not:** Call external APIs, initialize database connections at import time, or hard-code model IDs.
