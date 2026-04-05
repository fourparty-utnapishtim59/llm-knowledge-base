# Why not RAG?

Retrieval-Augmented Generation is the default answer to "how do I give my LLM access to a large document corpus." The recipe is well-established: chunk documents, embed them into vector space, store them in a specialised database, run similarity search at query time, stuff the retrieved chunks into the context window.

This schema doesn't use that stack. Here's why — and when you should use RAG instead.

---

## What RAG is optimised for

RAG was designed for a specific problem: giving a language model access to a corpus too large to fit in a context window, with low-latency retrieval at query time. It solves that problem well. If you have millions of documents, heterogeneous content types, and need sub-second responses, RAG is the right architecture.

The issues arise when RAG is applied to corpora where it isn't the right fit — which includes most personal and team knowledge bases.

---

## The compiled wiki approach

Instead of chunking and embedding documents, the compiled wiki approach does something different: an LLM reads the raw source material and **writes a structured knowledge base** from it. Concept articles. Summaries. Backlinks. Index files.

The LLM is not a retriever — it is the primary author and editor of the knowledge base. The knowledge base itself is the retrieval layer.

At query time, the agent reads the lightweight index files (`_index.md`, `_concepts.md`, `_graph.md`) to identify which full articles are relevant, then loads only those. This is structured navigation, not similarity search.

---

## Side-by-side comparison

| Dimension | Compiled wiki | Vector RAG |
|---|---|---|
| **Retrieval mechanism** | Index-first navigation + LLM reasoning | Embedding similarity search |
| **Knowledge representation** | Human-readable markdown articles | Opaque vector embeddings |
| **Transparency** | Every answer traces to a specific file | Chunk retrieval is a black box |
| **Editability** | Any text editor, any LLM | Requires re-embedding on update |
| **Contradiction handling** | Explicit linting workflow catches conflicts | Conflicting chunks silently co-exist |
| **Infrastructure** | Zero (just files) | Vector DB, embedding API, retrieval pipeline |
| **Cost** | LLM tokens only | LLM + embedding API + DB hosting |
| **Latency** | Single LLM call | Embedding call + vector search + LLM call |
| **Setup time** | Minutes | Hours to days |
| **Sweet spot** | 50–2,000 well-curated documents | 10,000+ heterogeneous documents |
| **Scales to** | ~2,000 articles before index navigation degrades | Millions of documents |

---

## Why RAG underperforms at personal knowledge base scale

**Chunking destroys structure.** RAG pipelines chunk documents into fixed-size fragments, often splitting sentences, paragraphs, or arguments mid-thought. The compiled wiki preserves the full structure of each concept — the LLM writes coherent articles, not arbitrary 512-token windows.

**Similarity search retrieves noise.** At small corpus sizes, embedding similarity tends to retrieve too many loosely related chunks. The compiled wiki's explicit graph of backlinks and cross-references is more precise than cosine similarity for a curated domain.

**Contradictions accumulate silently.** When two source documents make conflicting claims, a RAG system will retrieve both chunks and present both to the LLM without flagging the conflict. The compiled wiki's linting workflow explicitly scans for contradictions and quarantines conflicting articles until resolved.

**Chunks aren't inspectable.** When a RAG answer is wrong, it's hard to audit. You can't read a vector. With a compiled wiki, every answer cites specific files — you can open them in Obsidian and verify the source immediately.

**Setup cost is disproportionate.** For 100 articles, running a vector DB, managing an embedding pipeline, and handling re-indexing on updates is substantial overhead for marginal retrieval benefit over a well-structured index file.

---

## When to use RAG instead

Use RAG when:

- Your corpus exceeds ~2,000 documents and growing
- Documents are heterogeneous (PDFs, HTML, code, images, structured data) without a natural concept taxonomy
- You need sub-second retrieval latency
- The corpus changes frequently and re-compilation would be too slow
- Semantic similarity across the full corpus matters more than structured navigation (e.g. "find all documents that mention X" across a million documents)
- You're building a product where many users each need retrieval over a shared large corpus

Use the compiled wiki when:

- You have a focused research domain with 50–2,000 well-chosen sources
- You want to understand your corpus, not just retrieve from it
- Human-readable output and auditability matter
- You're working alone or in a small team
- You want the knowledge base to compound over time through linting and learning

---

## Hybrid approaches

The two aren't mutually exclusive. A common pattern as a wiki grows:

1. Start with the compiled wiki approach (zero infrastructure, fast to start)
2. Add a local vector index (ChromaDB + Nomic Embed via Ollama) when the article count exceeds ~500 and index navigation starts to feel slow
3. Use the vector index as a pre-filter — retrieve candidate article filenames, then load the full articles for the LLM to reason over
4. Keep the compiled wiki as the canonical knowledge store; the vector index is just a retrieval accelerator

The key insight: even in a hybrid, the LLM reasons over compiled wiki articles, not raw chunks. The structure and quality of the knowledge base matters more than the retrieval mechanism.

---

## The scale crossover point

Based on practical experience with this pattern, the crossover where RAG starts to outperform the compiled wiki approach is approximately:

- **Article count:** ~1,000–2,000 articles, where `_index.md` becomes too long for efficient navigation
- **Token cost:** when loading the full index files plus relevant articles regularly exceeds context window limits
- **Update frequency:** when new raw documents arrive faster than the LLM can compile them incrementally

Below that threshold, the compiled wiki is simpler, cheaper, more transparent, and produces better answers.

---

*For more on the retrieval design, see `AGENTS.md §5` (index maintenance) and `§7` (query workflow).*
