# Phase 5: Analytics Engineering

## Exit criteria
dbt builds against fixture Parquet, marts and data tests pass, analytics exports to Postgres, and `/trending` uses the mart.

## T5.1 dbt project and sources
- **Goal:** Define a dbt-duckdb project reading versioned lake Parquet.
- **Files:** Create `dbt_project.yml`, profiles example, sources, staging models, macros, schema YAML, and fixture data.
- **Approach:** Register raw partitions as sources; normalize dates/types; avoid loading article text into serving outputs; make local and CI paths configurable.
- **Dependencies:** T1.3 and Phase 0; Parquet schema versions must be settled.
- **Acceptance:** `dbt debug` with fixture profile passes; `dbt build --select staging` reads fixture files; source freshness/schema checks are documented.
- **Tests:** dbt source, schema, and fixture build tests.
- **Risks/unknowns:** dbt-duckdb versions, S3 credentials, and source path/version strategy are unspecified.
- **Do not:** Add warehouse-specific SQL that prevents DuckDB local execution.

## T5.2 Marts and data tests
- **Goal:** Build the four documented marts and enforce correctness.
- **Files:** Create models for daily mentions, co-occurrence, entities/sources, trending; schema/tests/docs.
- **Approach:** Define date grain and deduplication; calculate trailing 14-day z-score with explicit zero/stddev behavior; use stable entity IDs; add uniqueness, not-null, relationships, accepted-values tests.
- **Dependencies:** T5.1 and source schemas.
- **Acceptance:** `dbt build` passes fixtures; model columns/grains match `docs/DATA_MODEL.md`; edge cases for no baseline/zero variance have deterministic output; test failures fail build.
- **Tests:** Fixture-based SQL/data tests and expected-output comparisons.
- **Risks/unknowns:** Trending score formula details, timezone, and behavior with sparse history are ambiguous.
- **Do not:** Invent a ranking formula without documenting it and updating open questions/decisions.

## T5.3 Postgres export and API integration
- **Goal:** Export marts to `analytics` schema and make `/trending` read them.
- **Files:** Export job/models, migration, API repository/query, integration tests, and runbook.
- **Approach:** Define refresh/upsert strategy and ownership; validate mart rows before export; expose bounded date/pagination query; preserve Postgres canonical core separation.
- **Dependencies:** T5.2, T2.2, T1.1.
- **Acceptance:** Fixture build exports analytics rows; `/trending` returns expected ranked entities; refresh is rerunnable; failed export leaves previous valid snapshot intact if atomic swap is selected.
- **Tests:** Export idempotency/transaction tests and API contract tests.
- **Risks/unknowns:** Export mechanism, refresh cadence, and atomicity are unspecified.
- **Do not:** Make API calculate the mart ad hoc or write analytics rows into canonical tables.

## T5.4 CI and generated docs
- **Goal:** Run dbt against fixture Parquet in CI and publish accurate dbt docs artifacts.
- **Files:** CI workflow, fixture profile, artifact config, and docs.
- **Approach:** Use isolated fixture paths; run deps/seed/build/test/docs as applicable; retain manifest/catalog; avoid production credentials.
- **Dependencies:** T5.1-T5.3.
- **Acceptance:** CI `dbt build` passes without cloud access; intentional fixture violation fails; docs artifact lists sources/models/tests.
- **Tests:** CI dry run and negative data test.
- **Risks/unknowns:** dbt docs hosting and CI runtime are not specified.
- **Do not:** Query production lake in pull-request CI.
