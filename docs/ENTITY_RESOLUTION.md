# Entity resolution (mention -> Wikidata QID)

## Pipeline
1. Normalize mention (casing, whitespace, possessives). Merge same-surface mentions per article.
2. Candidate generation: Wikidata wbsearchentities (top 10) + local alias cache in Postgres.
   Cache every API response by (surface, type) with TTL; respect rate limits and set a User-Agent.
3. Type filter: map spaCy labels to allowed Wikidata classes (P31/P279 closure), e.g.
   PERSON -> Q5, ORG -> Q43229, GPE -> Q6256 or Q486972. Drop mismatched candidates.
4. Scoring features: string similarity, alias exact match, candidate popularity
   (sitelink count), embedding similarity between article context window and
   candidate description, co-occurring already-resolved entities in same article.
5. Decision: link if score >= T_high; NIL if best < T_low; between -> optional LLM
   adjudication over top-3 candidates with a JSON-schema answer. Thresholds in config.
6. NIL handling: create local entity (is_nil=true); cluster NILs by normalized name+type
   so repeated unknown entities converge. Re-attempt NIL resolution weekly.
7. Persist er_score and er_method on every mention for auditability.

## Evaluation (mandatory, lives in evals/er/)
- Hand-label 300+ mentions across 60+ articles (mix of types, ambiguous names like
  "Washington", "Apple", "Jordan"). Store as JSONL.
- Report precision, recall, F1 for linking; NIL precision/recall; per-type breakdown.
- Every ER change must report metrics before/after. Track in docs/DECISIONS.md.
- Known hard cases go in evals/er/hard_cases.jsonl and never get removed.
