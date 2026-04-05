# Contamination mitigation

When an LLM writes and maintains your knowledge base, a new failure mode emerges that doesn't exist in human-curated wikis: **contamination** — the gradual degradation of wiki quality through unchecked agent-generated content.

This doc describes how contamination happens, what it looks like, and the concrete patterns this schema uses to prevent it.

---

## What contamination looks like

Contamination is not dramatic. It doesn't look like obviously wrong information. It looks like:

- A concept article that confidently synthesises two sources that actually contradict each other, with no flag
- A summary that accurately reflects one paper but generalises it to a claim the paper doesn't support
- A backlink that's plausible but wrong — connecting two concepts that aren't actually related in the way implied
- An article written by the agent during a speculative query that gets filed back into the wiki as if it were well-sourced
- Confidence levels that drift upward over time as the agent cites its own previous articles as sources

None of these are obvious errors. They're plausible content that gradually erodes the reliability of the knowledge base. A linting pass months later may catch some of them — but by then, the contaminated articles have been cited in new articles, spreading the error.

---

## The core problem: LLMs are confident

Language models don't naturally express uncertainty. When asked to write a concept article, a model will produce fluent, confident prose regardless of whether the source material is thin, contradictory, or absent. This is useful for generating readable content — it's a problem when that content enters a knowledge base as authoritative.

The schema addresses this through structural constraints rather than prompt engineering. You can't reliably instruct a model to "be uncertain when appropriate." You can build a schema that makes the uncertainty visible and traceable.

---

## The sandbox-first model

The primary contamination control in this schema is **sandbox-first promotion**.

The agent writes freely to two directories:
- `output/` — query results, reports, slides, figures
- `learning/` — flashcards, review queue, gap tracking

It writes to `wiki/` only when it has met the quality rules in `AGENTS.md §11` or received explicit instruction from the human.

Think of it like a data pipeline:

```
raw/          →   output/     →   wiki/
(immutable)       (staging)       (production)
```

Content in `output/` is an artifact of a query session. It's useful, readable, and often accurate — but it hasn't been validated. Promotion to `wiki/` is a deliberate step.

The agent marks every output report with `filed_back: false` by default. When a query produces a finding worth incorporating into the wiki, the agent explicitly lists which wiki files to update and waits for confirmation (or instruction to proceed) before writing to `wiki/`.

---

## The clean vault / messy vault pattern

For users who want a stricter separation, the schema supports a two-vault model popularised by Obsidian's co-creator:

**Messy vault** — the agent's working environment. Contains `raw/`, `output/`, `learning/`, and draft wiki articles. The agent writes here freely. This is where exploration happens.

**Clean vault** — the curated knowledge base. Contains only `wiki/` articles that have been explicitly promoted. The human reviews and approves each promotion. This vault is authoritative.

In Obsidian, these are two separate vaults. The agent operates on the messy vault. The clean vault is the one you trust, share, and build on for high-stakes use.

To implement this, add to `AGENTS.md`:

```markdown
## Vault model

This repo uses a two-vault model:
- `messy/` — agent working environment, free writes permitted
- `clean/` — curated wiki, promotion requires explicit human approval

The agent never writes directly to `clean/`. It proposes promotions by appending to `output/promotion-queue.md`.
```

---

## Confidence levels

Every wiki article carries a `confidence` field in its frontmatter:

```yaml
confidence: high       # high | medium | low | speculative
```

The agent assigns confidence based on sourcing:

| Level | Criteria |
|---|---|
| `high` | 2+ primary sources, no contradictions detected, recently linted |
| `medium` | 1–2 sources, or sources older than 6 months, or minor ambiguities |
| `low` | Single source, or thin coverage, or imputed from web search |
| `speculative` | Agent-inferred, no direct source, or contradictions unresolved |

**Speculative articles are not linked into the graph.** The `_graph.md` adjacency list only includes articles with `confidence: medium` or above. This prevents speculative content from propagating through backlinks into otherwise well-sourced articles.

The linting workflow audits confidence levels and downgrades articles where:
- Sources have become stale (>6 months old for fast-moving domains)
- The article has fewer than 2 sources
- A contradiction has been detected

Confidence can only be **upgraded** when new sources are added or contradictions resolved — never automatically.

---

## Quarantine

When a contradiction is detected between two articles or between an article and a source, the linting workflow adds a quarantine flag:

```yaml
status: quarantined
quarantine_reason: "Contradicts claims in summaries/source-b.md re: claim X"
quarantine_date: YYYY-MM-DD
```

Quarantined articles:
- Are excluded from `_graph.md` until the contradiction is resolved
- Are marked visibly in `_index.md` with a `[quarantined]` tag
- Are not used as sources for new article synthesis
- Appear as the first items in the next linting report

Resolution options:
1. The human reviews both sources and decides which is correct → agent updates the article and removes the flag
2. The contradiction is real and both claims are valid in different contexts → agent splits the article or adds a nuance section
3. One source supersedes the other → agent updates confidence and provenance

---

## Citation traceability

Every claim in a wiki article must trace to a file in `raw/` or be explicitly marked:

```yaml
source: web-imputed     # filled in by agent using web search during linting
source: agent-inferred  # logical inference from other wiki articles, no direct source
```

The agent never writes claims without one of:
- A citation to a `summaries/` file (which traces back to `raw/`)
- A `source: web-imputed` marker with a note on the search query used
- A `source: agent-inferred` marker with the reasoning chain noted

This makes every article auditable. When you read a claim and want to verify it, you can follow the citation chain from the wiki article back to the original raw source.

---

## Output file-back discipline

The most common contamination path is this: you run a query, get a useful answer, and ask the agent to file it back into the wiki. The answer was generated by the LLM reasoning over wiki articles — it's not a new source, it's a synthesis. If it gets filed as an article with `confidence: high`, you've created a circular citation: the wiki citing itself.

The schema prevents this through the `filed_back` and `filed_back_to` fields in output reports:

```yaml
filed_back: true
filed_back_to:
  - concepts/concept-a.md   # added one paragraph under "Further context"
  - topics/topic-b.md       # added to open questions section
```

When a query output is filed back, the agent:
1. Marks the source as `source: agent-inferred` in the updated wiki article
2. Sets `confidence` to no higher than `medium` regardless of the agent's confidence in the answer
3. Adds a note: "This section synthesised from query output YYYY-MM-DD-slug.md — not directly sourced"

Agent-inferred content is useful. It becomes contamination only when it's treated as equivalent to sourced content.

---

## Linting as a control loop

Contamination mitigation is not a one-time setup — it's an ongoing control loop. The linting workflow (AGENTS.md §8) is the mechanism that catches drift before it compounds:

- **Contradiction scan** catches conflicting claims that slipped through during compilation
- **Confidence audit** catches articles that looked well-sourced initially but whose sources have become stale
- **Orphan detection** catches articles that got disconnected from the graph, often a sign of a botched update
- **Gap detection** catches claims that reference concepts that were never properly sourced

Run linting regularly — at minimum, every time the wiki doubles in size. For active research, weekly linting passes are practical.

---

## Summary: the contamination controls

| Control | What it prevents |
|---|---|
| Sandbox-first promotion | Unreviewed query artifacts entering the wiki |
| Two-vault model (optional) | Any agent write to the authoritative vault without human approval |
| Confidence levels | Thin or speculative content being treated as authoritative |
| Speculative articles excluded from graph | Speculation propagating through backlinks |
| Quarantine flag | Contradictions spreading into new articles |
| Citation traceability | Unsourced claims |
| Filed-back discipline | Circular citations (wiki citing itself) |
| Regular linting | All of the above, on a recurring basis |

None of these controls are perfect in isolation. Together, they make contamination visible and reversible — which is the realistic goal. A compiled wiki will contain errors. The schema is designed so that when errors are found, they can be traced, quarantined, and corrected without corrupting everything that cited them.

---

*For the full quality rules spec, see `AGENTS.md §11`.*
*For the linting workflow, see `AGENTS.md §8`.*
*For the sandbox promotion model, see `AGENTS.md §12`.*
