# Phase 6: GraphRAG and Evaluations

## Exit criteria
Vector/graph retrieval, cited `/ask`, web Ask UI, and human-labeled evaluation metrics are implemented and tracked over at least three iterations.

## T6.1 Chunk and embed
- **Goal:** Generate versioned 384-dimensional article/entity embeddings without storing article text in serving systems.
- **Files:** Create chunker/embedder, model config, migrations/indexes, cache/usage logging, and tests.
- **Approach:** Process permitted text from private `lake/private/article_text/`; persist only versioned chunk/entity identity, 384-dimensional embedding, and `embedding_model`; configure `EMBEDDING_MODEL`/`EMBEDDING_DIM`; batch/retry/cache by content hash; create HNSW indexes; run the required bake-off against an API embedding model.
- **Dependencies:** T1.1/T1.2/T1.6, ADR-009, verified provider/model/pricing.
- **Acceptance:** Synthetic or license-clear fixture produces deterministic chunk IDs and 384-dimensional vectors with `embedding_model`; rerun is idempotent/cacheable; HNSW query returns IDs; no chunk text is persisted or returned; bake-off results are recorded.
- **Tests:** Chunk boundaries, dimension, cache, provider retry, migration/index, and prohibited-storage tests.
- **Risks/unknowns:** Model ID/dimension/pricing, chunk policy, and pgvector index tuning are unverified.
- **Do not:** Store raw chunks or cite embeddings as article content.

## T6.2 Retriever and graph expansion
- **Goal:** Combine vector retrieval, 1-2 hop graph expansion, and reranking into bounded context.
- **Files:** Create retriever/reranker/context schemas, query tests, and metrics.
- **Approach:** Embed query; retrieve article IDs; expand graph with depth/row limits; rerank using documented deterministic/configured method; attach source IDs/URLs and derived summaries; enforce context budget.
- **Dependencies:** T1.7, T6.1, T2.1 data access.
- **Acceptance:** Fixture query returns stable top-k IDs and graph facts; depth/context limits are enforced; unavailable vector/graph service produces typed failure; every context item has an ID.
- **Tests:** Recall fixture, ranking tie behavior, bounds, missing data, and injection/content policy tests.
- **Risks/unknowns:** Reranker/model choice, top-k, context budget, and graph fact serialization are unspecified.
- **Do not:** Pass private article text to the UI or allow unbounded graph expansion.

## T6.3 `/ask`
- **Goal:** Serve cited GraphRAG answers with refusal behavior for unsupported claims.
- **Files:** API route/service/schemas, prompt version, provider adapter reuse, cache/usage logging, and tests.
- **Approach:** Retrieve bounded context; prompt model to cite IDs and refuse unsupported claims; validate answer/citations schema; map IDs to URLs; handle provider/retrieval failure and cache safely.
- **Dependencies:** T6.2, T1.6, T2.4; model/pricing verification.
- **Acceptance:** Synthetic or license-clear fixture question returns answer with valid article citations; unsupported question returns documented refusal; fabricated citation/unknown ID is rejected; response contains no article text.
- **Tests:** Citation precision/schema, refusal, unknown facts, timeout/retry, cache, and prohibited-content tests.
- **Risks/unknowns:** Answer schema, refusal wording, judge model, and citation mapping are not fully defined.
- **Do not:** Present model knowledge as graph knowledge or return uncited factual claims.

## T6.4 RAG evaluation
- **Goal:** Evaluate retrieval and answer quality using 60+ human-labeled questions.
- **Files:** Create `evals/rag/` JSONL schema, runner, metrics, reports, fixtures, and docs.
- **Approach:** Define gold article IDs/facts; calculate recall@k, citation precision, faithfulness; separate deterministic retrieval from model judge; spot-check judge labels; version datasets and prompts.
- **Dependencies:** T6.2/T6.3 and human labeling manual task.
- **Acceptance:** Runner emits reproducible metrics and per-question failures; dataset has at least 60 questions with gold labels; three iteration reports are comparable and recorded in decisions/docs.
- **Tests:** Metric unit tests, malformed dataset validation, deterministic fixture run, and judge failure handling.
- **Risks/unknowns:** Gold-set scope, judge model/cost, faithfulness definition, and statistical reporting are unspecified.
- **Do not:** Tune and evaluate on the same hidden labels without recording the split.

## T6.5 Ask UI
- **Goal:** Add web question UI displaying answers and linked citations.
- **Files:** Create Ask route/components/client calls/generated types, states, and tests.
- **Approach:** Regenerate client from OpenAPI; render answer/citations only; show refusal/errors/loading; link URLs safely; preserve mobile accessibility.
- **Dependencies:** T6.3 and T2.5 client pipeline.
- **Acceptance:** Fixture question renders answer and citation links; refusal renders clearly; unknown citations cannot become links; no article body is displayed.
- **Tests:** Component, accessibility, Playwright, and citation-link security tests.
- **Risks/unknowns:** Streaming vs non-streaming response and UI answer limits are unspecified.
- **Do not:** Add client-side retrieval, hidden article scraping, or uncited claims.
