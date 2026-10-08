# Data model

## Postgres (canonical) — schema `core`
- sources(id, name, feed_url, kind[rss|gdelt], enabled)
- articles(id, source_id, url_canonical UNIQUE, title, published_at, fetched_at,
  content_hash, lang, status[fetched|ner_done|er_done|extracted|failed])
- mentions(id, article_id, surface, label[PERSON|ORG|GPE|...], start_char, end_char,
  sentence_hash, entity_id NULL, er_score NULL, er_method NULL)
- entities(id, wikidata_qid UNIQUE NULL, canonical_name, type, description,
  is_nil bool, aliases text[], created_at)  -- NIL entities are local, unresolved
- events(id, article_id, event_type, summary, occurred_at NULL, confidence)
- event_participants(event_id, entity_id, role)
- relations(id, subject_id, predicate, object_id, article_id, confidence, UNIQUE(subject_id,predicate,object_id,article_id))
- article_chunks_v1(id, article_id, chunk_idx, embedding vector(384), embedding_model)
- entity_embeddings_v1(entity_id, embedding vector(384), embedding_model)
- pipeline_runs(id, run_date, task, started_at, finished_at, status, rows_in, rows_out, meta jsonb)
- llm_calls(id, task, model, prompt_hash, tokens_in, tokens_out, cost_usd, cached bool, created_at)

Indexes: HNSW on embeddings; btree on articles(published_at), mentions(entity_id),
relations(subject_id), relations(object_id); GIN on entities.aliases.

Note: article_chunks store embeddings, not text. Retrieval returns article IDs/URLs;
snippets shown to users must be model-generated summaries, not verbatim text.

## Neo4j projection
Nodes: (:Entity {id, qid, name, type}), (:Article {id, url, title, published_at, source}),
(:Event {id, type, summary})
Rels: (Entity)-[:MENTIONED_IN {count}]->(Article), (Entity)-[:PARTICIPATES_IN {role}]->(Event),
(Event)-[:REPORTED_IN]->(Article), (Entity)-[:RELATED_TO {predicate, weight}]->(Entity),
(Entity)-[:CO_OCCURS_WITH {weight, last_seen}]->(Entity)
Constraints: unique Entity.id, Article.id, Event.id.
Sizing: design for the AuraDB Free lower bound of 50,000 nodes / 175,000 relationships
(TODO(verify) actual limits in the Aura console when creating the instance;
`NEO4J_MAX_NODES`, `NEO4J_MAX_RELS`). Postgres retains full history. Neo4j contains
Entity and Event nodes, weighted entity-entity edges, and Article nodes only within the
rolling `ARTICLE_WINDOW_DAYS` window (default 14). Create `CO_OCCURS_WITH` only above
`CO_OCCUR_MIN_WEIGHT` and for top-K entities per day; prune by `last_seen`. Before
writing, the projector checks counts and fails soft at 90% of configured caps by
skipping lowest-priority edges and logging skipped counts. Paused instances receive
bounded retry for resume. See ADR-008.

## Lake (Parquet)
`lake/<layer>/<table>/date=D/run_id=R/*.parquet`, with active runs selected by
`lake/_manifests/<table>/date=D.json`. Immutable run outputs are cleaned up after a
configured `RUN_RETENTION_DAYS` (default 14); cleanup never deletes any run referenced
by a current manifest. Backfills older than 30 days cannot rerun steps requiring
article text. Permitted article text is separate private input under
`lake/private/article_text/`, expires after 30 days, and is never in serving tables.
Schemas are versioned and enforced with pandera. dbt reads active runs via manifests.

## dbt marts
- fct_entity_mentions_daily(entity_id, date, mention_count, article_count)
- fct_cooccurrence(entity_a, entity_b, date, weight)
- dim_entities, dim_sources
- mart_trending_entities(date, entity_id, score)  -- z-score vs trailing 14d baseline
