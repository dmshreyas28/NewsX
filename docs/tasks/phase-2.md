# Phase 2: API, Cache, Web, Instrumentation

## Exit criteria
The local stack supports search/entity/graph/trending/article flows, generated-client web navigation, and passing Playwright/contract tests.

## T2.0 API contract documentation
- **Goal:** Define the API contract in `docs/API.md` before endpoint implementation.
- **Files:** Create `docs/API.md`; reference it from Phase 2 task documentation.
- **Approach:** Specify route/method, request and response schemas, pagination, standard errors, cache TTLs, rate limits, and bounds for health, entity search/detail/neighbors, graph path, trending, and article metadata. Specify that responses never include article text. Define `/trending` OpenAPI schema per ADR-011, including fields, ordering, and the shared contract expected of both implementations.
- **Dependencies:** Phase 0 and ADR-011.
- **Acceptance:** `docs/API.md` defines all Phase 2 endpoints, schemas, pagination, error model, cache TTLs, rate limits, and the `/trending` contract; its examples are internally consistent and suitable to implement as OpenAPI models.
- **Tests:** Documentation review/check that every endpoint in `docs/ROADMAP.md` T2.2 and ADR-011 is specified; no runtime tests at this documentation task.
- **Risks/unknowns:** Product-level response fields, pagination details, and endpoint limits are not established in existing base docs and must be documented as decisions or explicit open questions.
- **Do not:** Implement API/application code or contradict privacy rules in `AGENTS.md` and `docs/ARCHITECTURE.md`.

## T2.1 FastAPI foundation
- **Goal:** Provide typed FastAPI app, health endpoint, settings, OpenTelemetry hooks, and OpenAPI export.
- **Files:** Create `api/` app, Pydantic request/response models, config, health route, instrumentation, tests, and OpenAPI generation command.
- **Approach:** Keep startup/lifespan explicit; inject Postgres/Neo4j/Redis clients; define error model; expose readiness separately from liveness; instrument without making telemetry mandatory locally.
- **Dependencies:** T2.0, Phase 0 and T1.1 schema; OTel exporter credentials from manual setup are optional.
- **Acceptance:** `make api` serves health; health/readiness return documented statuses; OpenAPI export is deterministic; invalid responses fail validation; `make check` passes.
- **Tests:** App/client health, dependency failure, schema, and OpenAPI snapshot tests.
- **Risks/unknowns:** OTel package/exporter choices and health semantics are unspecified.
- **Do not:** Hand-write the frontend API client or expose article text.

## T2.2 Domain endpoints
- **Goal:** Implement `/entities/search`, entity detail/neighbors, `/graph/path`, `/trending`, and `/articles/{id}`.
- **Files:** Create route/service/repository modules, schemas, query tests, and OpenAPI updates.
- **Approach:** Use Postgres as canonical for metadata and Neo4j for graph traversal where appropriate; add pagination and stable ordering; return model-generated summaries only; parameterize all queries.
- **Dependencies:** T2.0, T2.1, T1.1/T1.7. `/trending` uses the Phase 2 Postgres implementation; Phase 5 swaps only the implementation to the analytics mart.
- **Acceptance:** Contract fixtures return documented status/schema; nonexistent IDs return standard 404; pagination is stable; graph path bounds depth; article response omits text; queries are explainable and indexed.
- **Tests:** Unit repository tests, DB integration tests, contract tests, authorization/rate-limit preparation, and prohibited-field assertions.
- **Risks/unknowns:** Exact non-trending response shapes and graph path semantics remain underspecified; the `/trending` contract is fixed by ADR-011.
- **Do not:** Add auth/accounts or return raw snippets/text.

## T2.3 Redis caching
- **Goal:** Add cache layer with TTLs, invalidation strategy, and hit metrics.
- **Files:** Create cache abstraction, endpoint integration, config, metrics, and tests.
- **Approach:** Key by normalized request/version; serialize validated responses; set endpoint-specific TTLs from config; fail open on Redis errors; document invalidation on projection/data refresh.
- **Dependencies:** T2.0, T2.1/T2.2 and Redis service.
- **Acceptance:** Repeated identical request records a hit and avoids backend call; TTL expiration misses; Redis outage still returns backend response; metrics expose hit/miss counts.
- **Tests:** Key stability, TTL, serialization, outage, stampede-safe behavior if implemented.
- **Risks/unknowns:** TTL values, cache invalidation scope, and metrics backend are not specified.
- **Do not:** Cache secrets, unvalidated responses, or hide backend failures.

## T2.4 Rate limiting and pagination
- **Goal:** Add auth-free rate limiting, bounded pagination, and consistent errors.
- **Files:** Middleware/dependencies, pagination models, error handlers, config, tests, and docs.
- **Approach:** Choose a Redis-backed algorithm; apply per route/IP with documented limits; enforce max page/depth; use opaque cursor or stable offset consistently; return retry metadata.
- **Dependencies:** T2.0, T2.1/T2.3; limits are a product decision and require explicit configuration.
- **Acceptance:** Excess requests receive documented 429 response; limits are configurable; pagination cannot exceed bounds; all errors match one schema.
- **Tests:** Boundary, concurrent, Redis outage, cursor/order, and error serialization tests.
- **Risks/unknowns:** Limits, proxy IP trust, and deployment topology are unspecified.
- **Do not:** Claim security-grade identity controls for auth-free IP limiting.

## T2.5 Web explorer
- **Goal:** Build Next.js 15 TypeScript UI using the generated OpenAPI client for search, entity, and force-graph views.
- **Files:** Create `web/` pages/components/styles/generated client config, tests, and env example.
- **Approach:** Export OpenAPI; generate client in build; implement loading/error/empty states; render metadata and derived summaries only; keep graph depth/size bounded; support desktop/mobile.
- **Dependencies:** T2.0, T2.1/T2.2 and stable API schema.
- **Acceptance:** `make web` starts; search -> entity -> graph flow works against local API; client is generated rather than hand-written; mobile layout is usable; no article body appears in DOM/network UI data.
- **Tests:** Component tests, generated-client check, accessibility smoke, and responsive Playwright coverage.
- **Risks/unknowns:** API client generator, graph library, and visual design are unspecified.
- **Do not:** Add Ask UI before Phase 6 or invent API response fields.

## T2.6 E2E and contracts
- **Goal:** Verify API/web integration from search through graph.
- **Files:** Create contract fixtures, Playwright config/specs, test seed helpers, and CI workflow updates.
- **Approach:** Start local dependencies; seed deterministic records; run API contract assertions and browser flow; isolate external services; collect artifacts on failure.
- **Dependencies:** T2.0 and T2.1-T2.5.
- **Acceptance:** `make check` includes contract/e2e gates or clearly separates environment-dependent checks; search -> entity -> graph passes; malformed API schema fails test; CI artifacts are retained on failure.
- **Tests:** Required by the task itself, including mobile viewport and prohibited-content assertions.
- **Risks/unknowns:** Browser installation and CI service orchestration are unspecified.
- **Do not:** Use live news/provider/LLM calls in CI.
