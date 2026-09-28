# LLM usage spec

Model names are config (`config/models.yaml`), not hard-coded. Verify current model
IDs and pricing at implementation time (TODO(verify)).

## Event/relation extraction
- Input: article title, published date, and text truncated to token budget, plus the
  resolved entity list (id + name + type).
- Output: JSON validated against a Pydantic schema:
  events[{type, summary(<=30 words), participants[{entity_id, role}], occurred_at?, confidence}]
  relations[{subject_id, predicate, object_id, confidence}]
- Rules in prompt: use only provided entity IDs; predicate from a fixed enum
  (e.g. LEADS, MEMBER_OF, ACQUIRES, LOCATED_IN, SANCTIONS, PARTNERS_WITH, OPPOSES);
  omit anything not stated in the text; no outside knowledge.
- Temperature 0. On schema failure: one repair retry, then mark article `failed` with reason.
- Cache key: hash(prompt_version + model + input).

## Prompt management
Prompts live in pipeline/prompts/<name>.v<N>.md. Bumping a version is a DECISIONS.md entry
with eval results attached. Never edit a released version in place.

## GraphRAG answer prompt
- Context = retrieved article metadata + graph facts (entity relations/events), each
  with an ID. Answer must cite IDs; unsupported claims must be refused ("not in the data").
- Output includes citations[] mapping to article URLs.
