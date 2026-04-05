# The learning layer

Most knowledge base workflows stop at organisation. You collect sources, the LLM compiles them into structured articles, and you have a well-linked wiki you can query. That's useful — but it's not mastery.

The core insight from learning science is that collecting and organising information is not the same as knowing it. To actually master a domain, you need three things that a static wiki doesn't provide:

1. **Active recall** — retrieving knowledge from memory, not just recognising it when you see it
2. **Spaced repetition** — reviewing material at increasing intervals, right before you'd forget it
3. **Gap awareness** — knowing what you don't know, not just what you've filed

The learning layer in this schema adds all three on top of the compiled wiki, without requiring any external tools.

---

## How it works

The learning layer lives under `learning/` and consists of three components:

```
learning/
├── _review.md       ← spaced repetition queue
├── flashcards/      ← one file per concept, LLM-generated Q&A pairs
└── gaps.md          ← open questions and knowledge gaps
```

The LLM generates and maintains all of this automatically. Every time a concept article is created or substantially updated, the agent generates or refreshes the corresponding flashcard file. Every linting pass updates `gaps.md` with newly detected open questions.

---

## Flashcards

Each file in `learning/flashcards/` mirrors a concept article. The agent generates questions that test understanding at multiple levels:

**Definitional** — can you state the concept precisely?
> Q: What is the key difference between self-attention and cross-attention?
> A: Self-attention operates within a single sequence (each token attends to all others in the same sequence). Cross-attention operates across two sequences (each token in one sequence attends to tokens in another, as in encoder-decoder architectures).

**Mechanistic** — do you understand how it works?
> Q: What are the three learned projections in scaled dot-product attention?
> A: Query (Q), Key (K), and Value (V) — all linear transformations of the input embeddings.

**Relational** — do you understand how it connects to other concepts?
> Q: Why does multi-head attention use multiple attention heads rather than one large one?
> A: Multiple heads allow the model to attend to different representation subspaces simultaneously — one head might capture syntactic relationships while another captures semantic ones.

**Applied** — can you use the knowledge?
> Q: In a transformer encoder-decoder, which attention mechanism allows decoder tokens to condition on the full encoder output?
> A: Cross-attention — each decoder position attends to all encoder positions.

The agent generates questions grounded in the wiki articles, not in general knowledge. This means the flashcards test your understanding of *your* knowledge base, including domain-specific terminology, frameworks, and claims from your specific sources.

---

## Spaced repetition scheduling

The review queue in `learning/_review.md` tracks due dates for each concept using a simplified version of the FSRS (Free Spaced Repetition Scheduler) algorithm — the same algorithm used by Anki and other modern spaced repetition tools.

### The algorithm

Each flashcard has two parameters:

- **Interval** (days until next review) — starts at 1 day for new cards
- **Ease** (multiplier applied to interval on correct recall) — starts at 2.5

After a review session, the agent updates these parameters based on your recalled performance:

| Result | Interval | Ease |
|---|---|---|
| Correct (easy) | interval × ease | ease + 0.1 (max 3.0) |
| Correct (good) | interval × ease | ease unchanged |
| Correct (hard) | interval × 1.2 | ease − 0.15 |
| Incorrect | reset to 1 day | ease − 0.2 (min 1.3) |

This means a card you find easy gets reviewed less and less often (interval grows quickly), while a card you consistently struggle with stays at short intervals until it sticks.

### Running a review session

Ask the agent:
```
Show me what's due in learning/_review.md and run a flashcard session.
```

The agent will:
1. Load `_review.md` and identify due cards
2. Present each question without the answer
3. Wait for your response
4. Evaluate your answer against the wiki article (not against general knowledge)
5. Ask you to rate your recall: again / hard / good / easy
6. Update the interval and ease in `_review.md`
7. Show you the correct answer and the source article

The LLM evaluates answers semantically — partial answers, paraphrases, and responses that capture the core idea all count as correct. You don't need to match the exact wording.

### What gets scheduled

By default, every concept article gets a flashcard file and enters the review queue. You can exclude concepts by adding `learning: false` to the article frontmatter.

For large wikis, you can focus the review queue on specific tags:
```
Run a flashcard session for concepts tagged [attention, transformers] only.
```

---

## Gap tracking

`learning/gaps.md` is maintained by the agent during linting passes and Q&A sessions. It tracks two types of gaps:

**Structural gaps** — concepts mentioned in articles that don't have their own entry:
```markdown
- [ ] Flash Attention — mentioned in summaries/flash-attention-paper.md and concepts/attention.md
  but no dedicated concept article exists. Relevant to: attention, memory efficiency.
```

**Epistemic gaps** — questions the wiki can't answer, detected when a query returns low-confidence results:
```markdown
- [ ] How does rotary position embedding (RoPE) compare to ALiBi at context lengths >32K?
  Source: agent-detected during query 2026-04-05. Relevant articles: concepts/positional-encoding.md
```

Gaps serve two purposes. First, they're a research agenda — a prioritised list of what to ingest next. Second, they're a metacognitive tool — knowing what you don't know is the first step to knowing it.

The agent sorts gaps by priority (number of articles that reference the missing concept) so the highest-value additions are always at the top.

---

## Socratic mode

Beyond scheduled flashcard review, you can ask the agent to quiz you on any topic interactively:

```
Quiz me on sequence modelling. Be Socratic — ask follow-up questions based on my answers.
Don't give me the answer until I've had at least one attempt.
```

In Socratic mode, the agent:
- Grounds all questions in the wiki (not general knowledge)
- Asks follow-up questions that probe your understanding of specific concepts
- Points you to the relevant wiki article when you're stuck rather than just giving the answer
- Identifies which concepts in your answer were correct and which were missing

This is useful for preparing for a presentation or publication, checking understanding before moving to a new topic, or stress-testing your model of a complex domain.

---

## Integration with Obsidian

The learning layer is fully viewable in Obsidian:

- `learning/_review.md` renders as a table you can scan at a glance
- `learning/flashcards/*.md` open as readable markdown files
- `learning/gaps.md` renders as a checklist you can tick off as you fill gaps

For a richer review experience, the [Obsidian Spaced Repetition plugin](https://github.com/st3v3nmw/obsidian-spaced-repetition) can read flashcard files directly from your vault using its native `#flashcard` syntax. To use it alongside this schema, ask the agent to add Obsidian SR syntax when generating flashcards:

```
Generate flashcards for concepts/transformer-attention.md using Obsidian Spaced Repetition syntax.
```

---

## Why this belongs in the schema

The learning layer is not an add-on. It's what separates a knowledge archive from a knowledge system.

Without active recall and spaced repetition, a compiled wiki tends to become a reference you occasionally search rather than knowledge you actually own. The linting workflow keeps the wiki accurate. The learning layer keeps you accurate — it ensures that what's in the wiki is also in your head, not just in your files.

The flashcards are grounded in your specific sources, your specific domain framing, and your specific open questions. That's different from a generic Anki deck or a NotebookLM flashcard session. The questions test your understanding of *your* compiled knowledge base — including the connections, contradictions, and gaps that only exist because of the specific sources you chose to ingest.

---

*For the schema spec of flashcard files and the review queue format, see `AGENTS.md §9`.*
*For the gap detection workflow, see `AGENTS.md §8` (linting).*
