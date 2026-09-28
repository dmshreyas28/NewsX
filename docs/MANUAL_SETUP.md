# Manual setup (human tasks — agents must not attempt these)

- [ ] GitHub repo, branch protection on main, require CI
- [ ] Neon project + pgvector extension enabled; connection string -> .env
- [ ] Neo4j AuraDB Free instance; note free-tier limits and inactivity pausing behavior
- [ ] LLM provider API key(s) (extraction + embeddings); set spend limits
- [ ] Upstash Redis (or local only until Phase 2)
- [ ] Vercel project linked to web/
- [ ] AWS account, MFA, IAM user/role for Terraform, AWS Budgets alert (Phase 4)
- [ ] Grafana Cloud free stack + OTLP credentials (Phase 2/7)
- [ ] GitHub Actions secrets for each of the above
- [ ] Label evals data: ER set (Phase 1), RAG set (Phase 6)
- [ ] Rewrite each DECISIONS.md "why" in your own words at phase end
