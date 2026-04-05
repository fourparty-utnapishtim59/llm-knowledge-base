---
query: ""                          # the question or task that generated this report
generated: YYYY-MM-DD
sources:                           # wiki files consulted
  - concepts/concept-a.md
  - topics/topic-a.md
filed_back: false                  # set to true if agent incorporates findings into wiki
filed_back_to: []                  # list of wiki files updated as a result
---

# Report — [Short title]

> Generated: YYYY-MM-DD
> Query: [original query]

## Answer

Direct answer to the query. No preamble.

## Detail

Supporting reasoning, evidence from the wiki, comparisons, caveats.

## Sources consulted

- [Article title](../wiki/concepts/slug.md) — what it contributed
- [Article title](../wiki/summaries/slug.md) — what it contributed

## Confidence

Overall confidence in this answer: high / medium / low

Note any claims that rest on `confidence: low` or `confidence: speculative` wiki articles.

## Suggested wiki updates

Optional. If this query revealed gaps, contradictions, or new connections worth filing back:

- [ ] Create concept article for X (currently mentioned in Y but has no entry)
- [ ] Update `concepts/z.md` — this report adds new evidence for claim X
- [ ] Add edge to `_graph.md`: concept-a → concept-b
