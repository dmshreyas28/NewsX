# Manual setup (human tasks — agents must not attempt these)

- [ ] GitHub repo
- [ ] After T0.3's first CI run passes on the combined output of T0.1 and T0.2
  (neither was run in CI), enable branch protection on main requiring the CI check.
  Do not require the CI check before this run passes.
- [ ] Neon project + pgvector extension enabled; connection string -> .env
- [ ] Neo4j AuraDB Free instance; note free-tier limits and inactivity pausing behavior.
  After creating the Aura instance, record the actual node and relationship limits shown
  in the console and set `NEO4J_MAX_NODES` and `NEO4J_MAX_RELS` accordingly; note the
  inactivity pause behavior shown.
- [ ] LLM provider API key(s) (extraction + embeddings); set spend limits
- [ ] Upstash Redis (or local only until Phase 2)
- [ ] Vercel project linked to web/
- [ ] AWS account, MFA, IAM user/role for Terraform, AWS Budgets alert (Phase 4)
- [ ] Grafana Cloud free stack + OTLP credentials (Phase 2/7)
- [ ] GitHub Actions secrets for each of the above
- [ ] Label evals data: ER set (Phase 1), RAG set (Phase 6)
- [ ] Rewrite each DECISIONS.md "why" in your own words at phase end
