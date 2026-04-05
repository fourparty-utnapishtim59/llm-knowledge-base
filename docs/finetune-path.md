# Finetune Path — Turning Your Wiki Into a Training Corpus

> Part of the [LLM Knowledge Base](https://github.com/arturseo-geo/llm-knowledge-base) schema.

---

## The idea

As your wiki grows and is repeatedly linted, it becomes a progressively cleaner, deduplicated, consistently-styled representation of your research domain. At that point it is more than a context store — it is a candidate training corpus.

## What you already have

If you've been running the schema, you have three natural sources of training data:

| Source | Format | Use case |
|---|---|---|
| `wiki/concepts/*.md` | Long-form articles with frontmatter | Instruction tuning — "explain X" → article body |
| `wiki/summaries/*.md` | Source document summaries | Summarisation fine-tuning |
| `learning/flashcards/*.md` | Structured Q&A pairs | Direct supervised fine-tuning on question→answer |

The flashcards are the highest-value asset. They are already in question→answer format, grounded in the wiki (not hallucinated), and cover your domain densely.

## Preparing the data

### Step 1: Export flashcards to JSONL

Convert `learning/flashcards/*.md` to JSONL format:

```jsonl
{"instruction": "What is RLHF?", "output": "Reinforcement Learning from Human Feedback..."}
{"instruction": "How does Constitutional AI differ from RLHF?", "output": "Constitutional AI replaces..."}
```

### Step 2: Export concept articles as instruction pairs

For each concept article, generate an instruction pair:

```jsonl
{"instruction": "Explain the concept of retrieval probability in GEO.", "output": "<full article body>"}
```

### Step 3: Deduplicate and validate

- Remove duplicate Q&A pairs
- Remove any flashcard where the answer contains "I don't know" or "speculative"
- Validate that all answers trace to a wiki article

## Recommended tools

| Tool | Purpose |
|---|---|
| [Distilabel](https://github.com/argilla-io/distilabel) | Synthetic data generation and AI feedback |
| [Unsloth](https://github.com/unslothai/unsloth) | Fast LoRA fine-tuning (2x speed, 60% less memory) |
| [Axolotl](https://github.com/OpenAccess-AI-Collective/axolotl) | Full-featured fine-tuning with multiple architectures |
| [LitGPT](https://github.com/Lightning-AI/litgpt) | Hackable LLM implementation for pretraining and fine-tuning |
| [MLX](https://github.com/ml-explore/mlx) | Apple Silicon fine-tuning |
| [Argilla](https://github.com/argilla-io/argilla) | Human-in-the-loop data curation |

## When to fine-tune

Fine-tuning makes sense when:

- Your wiki exceeds ~200 articles in a focused domain
- You need a smaller, faster model for repeated queries against the same knowledge
- You want to deploy a domain expert without sending your full wiki as context every time
- Your flashcard library exceeds ~500 Q&A pairs

Fine-tuning does **not** make sense when:

- Your wiki is still growing rapidly (the model will be outdated quickly)
- You need the model to reason about new, unseen information
- Your domain is broad rather than deep

## Quality threshold

Only fine-tune on articles with `confidence: established` or `confidence: supported`. Never include `speculative` or `quarantined` content in training data.

---

Built and maintained by **[Artur Ferreira](https://github.com/arturseo-geo)** · Part of the **[The GEO Lab](https://thegeolab.net)** toolkit.
