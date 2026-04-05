---
title: "RLHF"
type: concept
created: 2026-04-05
updated: 2026-04-05
confidence: high
sources:
  - summaries/concrete-problems-ai-safety.md
  - summaries/constitutional-ai-paper.md
related:
  - concepts/constitutional-ai.md
  - concepts/reward-hacking.md
  - concepts/scalable-oversight.md
  - topics/ai-alignment-overview.md
tags: [alignment, training, human-feedback]
---

# RLHF

Reinforcement Learning from Human Feedback is a training methodology that uses human preference comparisons to shape language model behaviour. Rather than optimising against a fixed objective function, RLHF learns what humans prefer by training a reward model on pairwise comparisons of model outputs, then uses that reward model as the optimisation target for fine-tuning via reinforcement learning.

## Core mechanism

RLHF proceeds in three stages:

**Stage 1 — Supervised fine-tuning (SFT):** The base model is fine-tuned on a dataset of human-written demonstrations of desired behaviour. This produces a model that is broadly in the right territory but not yet aligned to nuanced human preferences.

**Stage 2 — Reward model training:** Human labellers are shown pairs of model outputs and asked which they prefer. A separate reward model is trained to predict these preferences. The reward model learns to score outputs on a continuous scale that approximates human judgment.

**Stage 3 — RL fine-tuning:** The SFT model is further trained using the reward model as a signal. The policy (the language model) is optimised to maximise reward model scores while staying close to the SFT model via a KL-divergence penalty. The KL penalty prevents the model from finding degenerate strategies that score high on the reward model but produce nonsensical outputs.

## Key relationships

- **[Reward hacking](reward-hacking.md)** is a primary failure mode of RLHF. The reward model is an imperfect proxy for human preferences, and the RL stage can find ways to maximise the proxy that don't align with the underlying intent.
- **[Constitutional AI](constitutional-ai.md)** extends RLHF by replacing some human feedback with AI-generated feedback guided by a set of principles, reducing the human labelling burden and making the value system explicit.
- **[Scalable oversight](scalable-oversight.md)** addresses a fundamental limitation of RLHF: human labellers can only reliably evaluate outputs they can understand. Scalable oversight techniques aim to extend reliable supervision to tasks beyond direct human evaluation.

## Limitations

- Human labeller preferences are inconsistent, vary across individuals, and can be manipulated by superficial features (length, confidence of tone)
- The reward model is a lossy compression of human preferences and is susceptible to reward hacking at scale
- RLHF is expensive: it requires substantial human annotation and multiple training runs
- The value system is implicit in the labelling guidelines and labeller behaviour, making it hard to audit or adjust

## Open questions

- How does RLHF performance scale with the number of human comparisons?
- Can reward models trained on simple tasks generalise to evaluating complex, long-horizon outputs?

## Sources

- [Concrete Problems in AI Safety](../summaries/concrete-problems-ai-safety.md)
- [Constitutional AI Paper](../summaries/constitutional-ai-paper.md)
