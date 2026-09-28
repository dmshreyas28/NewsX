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
Sizing: verify current Aura Free node/relationship limits (TODO(verify)); design a
retention/pruning policy so the graph stays within limits (e.g. keep last N days of
Article nodes, keep aggregated edges).

## Lake (Parquet)
`lake/<layer>/<table>/date=D/run_id=R/*.parquet`, with active runs selected by
`lake/_manifests/<table>/date=D.json`. Immutable run outputs are cleaned up after a
configured number of days. Permitted article text is separate private input under
`lake/private/article_text/`, expires after 30 days, and is never in serving tables.
Schemas are versioned and enforced with pandera. dbt reads active runs via manifests.

## dbt marts
- fct_entity_mentions_daily(entity_id, date, mention_count, article_count)
- fct_cooccurrence(entity_a, entity_b, date, weight)
- dim_entities, dim_sources
- mart_trending_entities(date, entity_id, score)  -- z-score vs trailing 14d baseline
