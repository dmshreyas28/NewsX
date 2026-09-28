# AGENTS.md — rules for AI coding agents (Zed, opencode, any model)

Read this file, then docs/ROADMAP.md, before doing anything. Work on ONE task at a time.

## Ground rules
1. Only do the task you were given. Do not refactor unrelated code or add features.
2. Never invent APIs, model IDs, package versions, or free-tier limits. If unsure,
   say "unverified" in the PR/commit notes and leave a TODO(verify).
3. Never commit secrets. All config via env vars; keep .env.example current.
4. Every task ends with: tests pass, lint/type checks pass, docs updated if behavior changed.
5. Stop and report if a task's acceptance criteria are ambiguous or conflict with docs.
   Do not guess silently.
6. Do not copy code from third-party repos. Use libraries via their public APIs.
7. Article text is processing input only. Never store it in the graph, never return
   it from the API, never render it in the UI. Store URL, title, publisher, dates,
   and derived data (entities, events, short model-generated summaries).

## Conventions
- Python 3.11+, uv or pip-tools for deps, ruff (lint+format), mypy --strict on new code,
  pytest. Pydantic v2 models at every boundary. Type hints everywhere.
- TypeScript strict mode, ESLint, Prettier. API client generated from the OpenAPI spec,
  never hand-written.
- SQL migrations are forward-only files in db/migrations/ (NNNN_description.sql).
- Pipeline tasks are idempotent and re-runnable for any date (backfill safe).
  Use natural keys / upserts, never blind inserts.
- Structured JSON logging. Every pipeline task logs counts in/out and duration.
- LLM calls: config-driven model name, temperature 0 for extraction, JSON-schema
  validated output, retry with backoff, cache by content hash, log token usage.
- Commit style: Conventional Commits. One task = one branch = one PR.

## Definition of done (per task)
- Acceptance criteria in docs/ROADMAP.md for the task are met and demonstrable.
- New code has tests (unit; integration where a DB is touched).
- `make check` passes (ruff, mypy, pytest, eslint, tsc as applicable).
- docs/DECISIONS.md updated if a design choice was made or changed.
- A short "How I verified" note in the PR description with commands run.

## Commands (keep this section current)
- make check | make test | make migrate | make ingest-sample | make api | make web
