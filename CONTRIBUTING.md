# Contributing

This repo is a community-maintained schema standard. Contributions are welcome — the goal is a spec that works across different domains, LLMs, and toolchains, not one that optimises for a single person's workflow.

---

## What we're looking for

**High value:**
- Worked example wikis in `examples/` on different research domains
- Improvements to the learning layer (alternative scheduling algorithms, better flashcard formats)
- Additional output format templates (reveal.js, Jupyter, HTML)
- CLI tooling that automates linting or compilation
- Field reports: what broke, what scaled, what didn't

**Medium value:**
- Clarifications to `AGENTS.md` where the spec is ambiguous
- New optional frontmatter fields with clear use cases
- Additional docs under `docs/`

**Low value (please discuss in an issue first):**
- Breaking changes to directory layout or required frontmatter fields
- Renaming existing index files
- Changing the version numbering scheme

---

## Schema change process

Changes to `AGENTS.md` follow semantic versioning (see `CHANGELOG.md`):

- **Patch** (e.g. 1.0.1): clarifications only, no agent behaviour changes. Submit a PR directly.
- **Minor** (e.g. 1.1.0): new optional fields or workflow steps. Open an issue first, describe the use case, wait for one approval before submitting a PR.
- **Major** (e.g. 2.0.0): breaking changes to layout or required fields. Must go through an issue with community discussion. Breaking changes need a migration guide in `docs/`.

Every schema change must:
1. Update the version header in `AGENTS.md`
2. Add an entry to `CHANGELOG.md`
3. Update any affected templates in `templates/`
4. Update any affected example files in `examples/`

---

## Submitting a PR

1. Fork the repo
2. Create a branch: `git checkout -b your-feature-or-fix`
3. Make your changes
4. Check that all templates still validate against the schema in `AGENTS.md`
5. Open a PR with a clear description of what changed and why

PR titles should follow the pattern:
- `fix: clarify confidence field behaviour in §4`
- `feat: add topic.md template`
- `docs: add worked example wiki for machine learning`
- `schema: add optional summary_sentence field to concept frontmatter [minor]`

---

## Adding an example wiki

The most useful contribution is a worked example in `examples/`. To add one:

1. Create a directory: `examples/your-topic/`
2. Follow the full directory layout from `AGENTS.md §1`
3. Include at least 5 concept articles, 3 summaries, a populated `_index.md`, `_concepts.md`, and `_graph.md`
4. Include at least 3 flashcard files in `learning/flashcards/`
5. Add a short `examples/your-topic/README.md` explaining the domain and any domain-specific schema adaptations
6. The example should be LLM-compiled, not hand-written — note which model compiled it

---

## Reporting issues

Use the issue templates in `.github/ISSUE_TEMPLATE/`:

- **Bug report**: something in the schema is ambiguous, contradictory, or causes agent misbehaviour
- **Feature request**: a new workflow, field, or template you'd like to see
- **Example wiki request**: a domain you'd like a worked example for

---

## Code of conduct

Be direct and specific. This is a technical project — the best contributions are concrete, reproducible, and grounded in actual use. Vague "this should be better" feedback isn't actionable. "The agent consistently misinterprets §6 step 3 when the raw document is a PDF with no metadata" is.
