# Phase 7: Hardening and Portfolio

## Exit criteria
Operational dashboards/runbook, recorded load-test p95, and portfolio documentation with screenshots/metrics are complete.

## T7.1 Observability and runbook
- **Goal:** Provide Grafana dashboards, alerts, traces/metrics, and operational procedures.
- **Files:** Create dashboard definitions, alert rules, OTel configuration, `docs/RUNBOOK.md`, and deployment updates.
- **Approach:** Instrument API/pipeline metrics named in architecture; define labels/cardinality limits; alert on failures, latency, ER/NIL rate, LLM cost, cache health; document diagnosis and recovery.
- **Dependencies:** T0.4, T1.8, T2, T3, production choices.
- **Acceptance:** Dashboard displays fixture/staging metrics; alert rules validate; runbook covers failed pipeline, stale graph, API outage, cost spike, and rollback; traces correlate a request/task.
- **Tests:** Metric emission/unit tests, config validation, and tabletop runbook exercise.
- **Risks/unknowns:** Grafana Cloud plan/limits, OTel exporter, alert thresholds, and retention are unverified.
- **Do not:** Log raw article text, secrets, or high-cardinality unbounded IDs.

## T7.2 Load test
- **Goal:** Measure API performance and record p95 under representative load.
- **Files:** Create `load/` k6 or Locust scenarios, seeded-data instructions, and report.
- **Approach:** Select tool and workload; test health/search/entity/graph/ask separately; warm/cold cache cases; define concurrency, duration, SLO, and resource observations; repeat enough for stable comparison.
- **Dependencies:** T2/T6, deployed staging, observability.
- **Acceptance:** Load command is reproducible; report includes workload, environment, p50/p95/p99, errors, and cache state; bottlenecks and follow-up thresholds are documented.
- **Tests:** Scenario syntax and smoke run at low load.
- **Risks/unknowns:** Target traffic/SLO and representative dataset size are unspecified.
- **Do not:** Load production without approval or report synthetic numbers as real-world performance.

## T7.3 README and screenshots
- **Goal:** Document architecture, setup, usage, screenshots, and measured metrics accurately.
- **Files:** Modify `README.md`; add diagrams/screenshots under docs/assets; link runbook/evaluations.
- **Approach:** Update status from pre-implementation only as milestones are actually complete; include commands and limitations; label synthetic/staging data and metrics.
- **Dependencies:** All prior phases and T7.1/T7.2.
- **Acceptance:** Fresh user can follow setup; architecture diagram matches code; screenshots show current UI; metrics link to reports; no unverified claims remain unlabeled.
- **Tests:** Follow README in clean environment or record blockers; check links and image paths.
- **Risks/unknowns:** Screenshot environment and publication/privacy policy are unspecified.
- **Do not:** Claim production readiness, free-tier compliance, or evaluation quality without evidence.

## T7.4 Write-up
- **Goal:** Explain five key decisions, failures, and future changes for the portfolio.
- **Files:** Create `docs/WRITEUP.md` and update decision links.
- **Approach:** Use ADRs and evaluation/load reports; explain alternatives, evidence, trade-offs, failures, and quantified outcomes; separate verified facts from future work.
- **Dependencies:** ADRs resolved as applicable, T6.4, T7.2.
- **Acceptance:** Five decisions have evidence and consequences; at least one failure and corrective action are described; metrics are reproducible; open questions remain explicit.
- **Tests:** Link checker and reviewer checklist.
- **Risks/unknowns:** Human owner must rewrite decision rationale per manual setup.
- **Do not:** Fabricate metrics, user impact, costs, or external limits.
