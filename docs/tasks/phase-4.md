# Phase 4: Cloud and IaC

## Exit criteria
The API serves real data publicly, production scheduling is selected and documented, and teardown is documented/tested.

## T4.1 Terraform foundation
- **Goal:** Provision remote state, S3 lake, ECR, IAM, lifecycle rules, and environment separation.
- **Files:** Create `infra/` Terraform modules, backend/provider config, variables/examples, and docs.
- **Approach:** Define least-privilege roles, encrypted resources, lifecycle/retention, tagging, plan-time validation, and remote state bootstrap procedure; use modules without embedding secrets.
- **Dependencies:** ADR-005; AWS account/manual setup; retention and runtime decisions inform IAM.
- **Acceptance:** `terraform fmt -check`, `terraform validate`, and a credentials-free plan with test variables succeed; plan includes S3/ECR/IAM/lifecycle; no secret values in state inputs/examples.
- **Tests:** Terraform validation, policy assertions, and plan review in CI.
- **Risks/unknowns:** AWS region, account boundaries, state backend bootstrap, exact lifecycle, and cost estimates.
- **Do not:** Apply infrastructure from CI until approval workflow exists or commit credentials.

## T4.2 API deployment
- **Goal:** Deploy the API using the runtime selected by ADR-006 with managed secret injection.
- **Files:** API Docker/deployment config, Terraform module, health/readiness config, and runbook.
- **Approach:** Compare Lambda container/App Runner/Fargate on cost, cold start, and connection behavior; select explicitly; configure networking, scaling, logs, env references, and health checks; test against managed Postgres/Neo4j/Redis.
- **Dependencies:** T2, T4.1, ADR-006, manual services.
- **Acceptance:** Staging endpoint serves health and one read endpoint; secrets come from SSM/Secrets Manager; deployment rollback is documented; smoke test passes.
- **Tests:** Container, deployment plan, health, connection-pool, and smoke tests.
- **Risks/unknowns:** Current pricing, network access, managed service allowlists, and runtime limits are unverified.
- **Do not:** Put secrets in Terraform variables/state unnecessarily or expose database ports publicly.

## T4.3 Production lake and schedule
- **Goal:** Write production partitions to S3 and run the selected scheduled pipeline architecture.
- **Files:** Pipeline storage adapter, IAM policies, scheduler deployment/config, and runbook.
- **Approach:** Select EC2 compose, ECS scheduled task, or GitHub Actions based on ADR-007; preserve CLI reuse; use atomic/versioned object strategy; verify retries and observability.
- **Dependencies:** T3, T4.1, ADR-007, S3 credentials.
- **Acceptance:** A controlled production-like run writes one date partition; rerun does not duplicate data; scheduler records success/failure; least-privilege access works.
- **Tests:** S3 integration, idempotent rerun, scheduler smoke, and failure recovery tests.
- **Risks/unknowns:** S3 atomic overwrite, schedule host costs, Airflow persistence, and network topology are unverified.
- **Do not:** Run production ingestion without spend limits, retention, or alerting.

## T4.4 CD workflow
- **Goal:** Build/push images and run Terraform plan on PR, apply on main with approval.
- **Files:** GitHub workflows, IAM/OIDC config, image tags, and deployment docs.
- **Approach:** Use immutable commit tags; scan/build; PR plan artifact; protected apply environment; separate state/workspaces; restrict permissions.
- **Dependencies:** T4.1-T4.3 and GitHub branch protection/manual secrets.
- **Acceptance:** PR produces reviewed plan; main apply requires approval; image digest is deployed; failed deployment can roll back; workflow permissions are minimal.
- **Tests:** Workflow lint, plan-only dry run, and staging deployment smoke test.
- **Risks/unknowns:** OIDC setup, branch protection, registry retention, and approval rules are manual/unverified.
- **Do not:** Auto-apply arbitrary PR code to production.

## T4.5 Cost guardrails and teardown
- **Goal:** Alert on AWS spend and prove resources can be destroyed safely.
- **Files:** Terraform budget/alert resources, `docs/` teardown runbook, and verification scripts.
- **Approach:** Define budget threshold/recipients; label resources; document state backup and dependency order; test destroy in disposable environment; identify external managed services not destroyed by Terraform.
- **Dependencies:** T4.1-T4.4 and AWS Budgets/manual contacts.
- **Acceptance:** Budget alert resource is in plan; teardown steps are executable; disposable environment destroys without orphaned Terraform-managed resources; external services are clearly listed.
- **Tests:** Terraform plan/destroy smoke and runbook dry run.
- **Risks/unknowns:** Budget amount, alert delivery, and destroy behavior are not specified.
- **Do not:** Destroy shared production resources or assume managed external services are owned by this stack.
