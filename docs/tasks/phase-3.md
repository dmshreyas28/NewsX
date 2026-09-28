# Phase 3: Orchestration

## Exit criteria
Airflow imports the DAG, a scheduled local run succeeds, backfill/catchup works for seven days, and a failed task recovers on retry.

## T3.1 Airflow compose and DAG
- **Goal:** Add local Airflow LocalExecutor and `nightly_news_kg` wrapping existing pipeline entrypoints.
- **Files:** Modify compose; create `airflow/dags/nightly_news_kg.py`, Airflow config/dependency files, and DAG tests.
- **Approach:** Keep DAG thin; call CLI stages; define schedule, timezone, task order, connections/env wiring, and healthcheck; avoid business logic in DAG.
- **Dependencies:** T1.9, Phase 0 compose.
- **Acceptance:** Airflow parses/imports DAG; task graph matches stage order; scheduler/webserver health checks pass; manual run invokes same CLI entrypoints.
- **Tests:** DAG import, task IDs/dependencies, and mocked operator invocation.
- **Risks/unknowns:** Airflow version, schedule/timezone, executor resource needs, and secret backend are unspecified.
- **Do not:** Duplicate pipeline logic or run unbounded catchup by default.

## T3.2 Retries, SLAs, alerts, backfill
- **Goal:** Make failures recoverable and validate seven-day backfill/catchup.
- **Files:** Modify DAG/config/docs; create backfill tests and failure fixtures.
- **Approach:** Configure bounded retries/backoff, task timeouts, SLA/failure notification hooks, explicit catchup policy, and date propagation; run seven dates against fixtures; deliberately fail one task and verify retry.
- **Dependencies:** T3.1 and idempotent T1 stages.
- **Acceptance:** `airflow dags test nightly_news_kg <date>` completes for fixture date; seven-date backfill creates one safe partition/run per date; injected transient failure succeeds on retry; permanent failure alerts/marks DAG failed.
- **Tests:** Retry, timeout, date templating, backfill isolation, and alert callback tests.
- **Risks/unknowns:** SLA values, notification channel, and production scheduler environment are not defined.
- **Do not:** Retry non-idempotent writes blindly or hide permanent failures.

## T3.3 CI DAG checks
- **Goal:** Prevent invalid DAG imports and structural regressions.
- **Files:** Modify CI; create DAG validation tests and fixtures.
- **Approach:** Install minimal Airflow test dependencies or use a supported isolated check; import DAGs; assert no import errors, cycles, missing tasks, or unexpected external calls.
- **Dependencies:** T3.1.
- **Acceptance:** CI fails on syntax/import/dependency errors and passes current DAG; tests do not contact production services.
- **Tests:** All DAG tests and a negative fixture where practical.
- **Risks/unknowns:** Airflow dependency weight and supported Python versions.
- **Do not:** Make CI depend on a running scheduler or live credentials.
