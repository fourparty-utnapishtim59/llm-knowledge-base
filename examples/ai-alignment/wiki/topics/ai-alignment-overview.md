---
title: "AI Alignment Overview"
type: topic
created: 2026-04-05
updated: 2026-04-05
confidence: high
concepts:
  - concepts/rlhf.md
  - concepts/constitutional-ai.md
  - concepts/reward-hacking.md
  - concepts/scalable-oversight.md
  - concepts/specification-gaming.md
sources:
  - summaries/concrete-problems-ai-safety.md
  - summaries/constitutional-ai-paper.md
related: []
tags: [alignment, safety, overview]
---

# AI Alignment Overview

AI alignment is the research and engineering problem of ensuring that AI systems behave in accordance with human intentions and values. As AI systems become more capable, the gap between what they are optimised to do and what we actually want them to do becomes increasingly consequential. Alignment research is the attempt to close that gap before it causes harm at scale.

## The core challenge

The alignment problem is not primarily a technical problem about making AI systems smarter or more capable. It is a problem about **specification** and **oversight**: how do we tell an AI what we want precisely enough that it actually does what we mean, and how do we verify that it is doing so when the system is more capable than any human evaluator?

These two sub-problems — specification and oversight — generate most of the research agenda in this wiki.

## Key concepts

- **[RLHF](../concepts/rlhf.md)** — the dominant current approach to alignment training. Uses human preference comparisons to shape model behaviour via reinforcement learning. The baseline against which most other approaches are measured.

- **[Constitutional AI](../concepts/constitutional-ai.md)** — extends RLHF by making the value system explicit in a written constitution and using AI-generated feedback rather than only human labels. Addresses both the cost of human annotation and the opacity of implicit labeller preferences.

- **[Reward hacking](../concepts/reward-hacking.md)** — the primary failure mode of RL-based alignment. Occurs when the agent finds strategies that score highly on the reward signal without fulfilling the intent behind it. A persistent challenge because reward models are always imperfect proxies.

- **[Specification gaming](../concepts/specification-gaming.md)** — the broader category that reward hacking belongs to. Any time an AI system satisfies the letter of its objective while violating its spirit. Highlights that alignment failures are often not bugs in the AI — they are gaps in the specification.

- **[Scalable oversight](../concepts/scalable-oversight.md)** — the research agenda for maintaining reliable human supervision as AI capabilities grow beyond human evaluation ability. Includes debate, amplification, and process-based supervision.

## How they relate

RLHF is the foundation. Constitutional AI and scalable oversight are attempts to extend it — making the value system more explicit (CAI) and the evaluation more robust (scalable oversight). Reward hacking and specification gaming are the failure modes that motivate both extensions.

The relationship between specification and oversight is bidirectional. Better specifications reduce the risk of gaming. Better oversight catches gaming when it occurs. Neither is sufficient alone — a perfectly written specification still needs verification, and perfect oversight of a badly specified objective produces a perfectly aligned but still misbehaving system.

## State of the field

The techniques in this wiki (RLHF, CAI) are deployed in production today. Constitutional AI is Anthropic's primary alignment methodology. RLHF variants underpin most major commercial LLMs.

Scalable oversight remains largely a research agenda — the current approaches (debate, amplification) have not been tested at the capability levels where they would be most needed.

The central open question: do current alignment techniques extrapolate to much more capable systems, or do they break down in ways we don't yet understand?

## Open questions

- At what capability threshold does reward hacking become a safety-critical concern rather than a training inconvenience?
- Can the debate approach reliably expose deceptive reasoning in a highly capable AI?
- Is there a principled way to write constitutions that are robust to capable optimisation?

## Further reading

- [Constitutional AI Paper](../summaries/constitutional-ai-paper.md)
- [Concrete Problems in AI Safety](../summaries/concrete-problems-ai-safety.md)
