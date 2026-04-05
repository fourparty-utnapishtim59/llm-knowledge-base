---
type: lint-report
generated: YYYY-MM-DD
wiki_size:
  articles: 0
  concepts: 0
  summaries: 0
  topics: 0
issues_found: 0
issues_resolved: 0
---

# Lint report — YYYY-MM-DD

## Summary

| Check | Issues found | Resolved | Deferred |
|---|---|---|---|
| Orphan detection | 0 | 0 | 0 |
| Contradiction scan | 0 | 0 | 0 |
| Confidence audit | 0 | 0 | 0 |
| Gap detection | 0 | 0 | 0 |
| Web imputation | 0 | 0 | 0 |
| Connection suggestions | 0 | — | — |

---

## Orphans

Articles with no backlinks in `_graph.md`. These are isolated — no other article references them.

- [ ] `concepts/orphaned-concept.md` — suggested action: link from `topics/relevant-topic.md` or delete if superseded

---

## Contradictions

Claims that conflict across articles or sources.

- [ ] `concepts/concept-a.md` claims X; `summaries/source-b.md` claims not-X
  - Sources: [source-a](../summaries/source-a.md), [source-b](../summaries/source-b.md)
  - Recommended resolution: [agent's suggestion]
  - Status: quarantined — `concept-a.md` flagged `status: quarantined` pending resolution

---

## Confidence downgrades

Articles downgraded due to stale sources (>6 months) or insufficient sourcing (<2 sources).

- [ ] `concepts/concept-b.md` — downgraded high → medium (only 1 source, dated YYYY-MM-DD)

---

## Gaps

Concepts mentioned in articles but lacking their own entry in `wiki/concepts/`.

- [ ] **Concept X** — mentioned in `concepts/concept-a.md` and `topics/topic-b.md`, no article exists
  - Suggested action: create `concepts/concept-x.md`
  - Priority: high (mentioned in 3+ articles)

---

## Web imputation

Gaps filled using web search. All imputed content marked `source: web-imputed` in frontmatter.

- `concepts/concept-c.md` — added definition paragraph, source: web-imputed (search: "concept c definition")

---

## Connection suggestions

New edges proposed for `_graph.md` based on content analysis.

- [ ] `concept-a → concept-d` — both discuss mechanism X; linking would surface this for queries on X
- [ ] `topic-b → topic-c` — significant conceptual overlap; cross-reference would help navigation

---

## New article candidates

Concepts or topics that have enough source material to warrant a full article.

- [ ] **Topic: Y** — covered across 4+ summaries with no synthesising topic article
- [ ] **Concept: Z** — complex enough to need its own entry, currently only a bullet in `concepts/concept-a.md`

---

## Actions taken this run

- Updated `_graph.md`: added N edges
- Created N stub articles for high-priority gaps
- Quarantined N articles pending contradiction resolution
- Downgraded confidence on N articles
- Filed this report to `output/reports/YYYY-MM-DD-lint.md`
