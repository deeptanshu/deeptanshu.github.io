---
title: "When Small Models Are Better: A Peek into Geometry of Intelligence"
date: 2026-01-26T21:45:48-08:00
draft: false
category: "LLMs"
excerpt: "Let's take a local manifold walk to understand why small models are sometimes better, more resource responsible and won't burn a hole in your pocket."
tags: ["LLMs", "Transformers", "SLMs", "AI"]
---


### TL;DR
Most everyday prompts are **local manifold walks**: the meaning stays in the same semantic basin, you’re just changing *shape* (summary, tone, structure). Small models are great at this. Big models are best when the task demands **global consistency** and multi-hop synthesis.

---

We have a **"sledgehammer" problem** in AI.

We are using massive, 100B+ parameter models to fix typos, format markdown, and summarize transcripts—tasks that are closer to **doing long division with a supercomputer** than “needing intelligence.”

I have previously written about the frustrating experience of running into the **Token Wall** in Claude Code almost every session. To stay under rate limits and be resource-responsible, we need a routing framework based on the **topology of the task**.

---

## 1. The Geometry: Reshaping vs. Creating

To understand why small models (SLMs) excel at certain tasks, we have to look at the **latent space**.

Imagine all possible human thoughts as a topographic map (a manifold). Large models have the "depth" to fly across the entire map, connecting distant continents of logic. Small models, however, are masters of the **local neighborhood**.

### The “Local Manifold Walk”

When you ask a model to *"make this sound like a LinkedIn post,"* you aren't asking for a new discovery. You are asking for a **local manifold walk**.

The original meaning and the rewritten meaning occupy the same semantic basin.

In vector terms:

$$
\vec{v}_{input} \approx \vec{v}_{output}
$$

The task is simply to find a different set of tokens that project onto the same coordinate. Since the "map" is already built into the SLM’s embeddings, it doesn't need 100 layers to find the exit.

---

## 2. The SLM Framework: Reshape, Refine, Extract

If your query fits into one of these three buckets, route it to a local model (like *Llama 3.2 3B* or *Phi-3.5*) and save your Claude tokens for the hard stuff.

### A. Reshaping (Information Reorganization)

**Task:** Summaries, outlines, turning a CSV into a JSON.

**Why SLMs win:** This is **attention salience**. The model isn't learning new concepts; it’s just re-weighting the importance of tokens already in your prompt. It identifies the high-salience vectors and re-emits them in a cleaner order.

### B. Refining (Tone and Style)

**Task:** "Tighten this up," "Fix my grammar," "Make this more assertive."

**Why SLMs win:** This is a **projection** task. The model maps the input to a "style" subspace. This is "cheap" because the global semantic structure remains constant.

### C. Extracting (Pattern Snapping)

**Task:** "Give me the action items," "What was the decision made in this meeting?"

**Why SLMs win:** This is **classification + extraction**. The model has learned the "shape" of an action item. It simply scans the text until a span of tokens "snaps" into that latent pattern.

---

## 3. The “Token Wall”: When to Route Up

Small models fail when the geometry of the task requires **global consistency**.

Deep reasoning requires maintaining multiple **latent variables** across time. If a task requires 10 steps of logic, and each step has a 5% chance of drifting off the manifold, an SLM will be miles away from the truth by step 10.

Route to the big model when:

- The abstraction depth is high (requires "multiple hops" of logic).
- The task requires synthesis across distant concepts  
  (e.g., *"Write a poem about quantum physics in the style of a 1920s noir novel"*).
- Global state is critical  
  (e.g., *"Refactor this entire repository to use a different state management pattern"*).

### The Mental Equation for Routing

Before you hit "Enter" on a 1.2M context window query, run this heuristic:

$$
Difficulty \approx \Delta \text{Information} + \text{Global Dependency Depth}
$$

If you are just reshaping information that is already in the prompt, let the small model handle the "geometry." Save your "intelligence" budget for the tasks that require creation.
