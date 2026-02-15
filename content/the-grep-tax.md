---
title: "The Grep Tax: What 9,649 Experiments Reveal About How AI Agents Actually Use Context"
description: "New research shows file format barely matters for AI accuracy -- but model capability and context architecture create a 21-point gap. What this means for agent builders."
date: "2026-02-10T12:00:00-05:00"
author: "ekkOS Team"
tags: ["research-commentary", "context-engineering", "ai-agents", "benchmarks"]
image: "/images/blog/the-grep-tax.png"
imageAlt: "A magnifying glass hovering over layers of file formats -- YAML, JSON, Markdown -- with some paths glowing efficiently and others tangled in loops"
draft: true
---

A new paper from Damon McMillan -- "Structured Context Engineering for File-Native Agentic Systems" -- presents one of the most comprehensive empirical studies on how LLM agents actually consume structured data. The scale is unusual for this kind of work: 9,649 experiments across 11 models, multiple file formats, and SQL schemas ranging from 10 to 10,000 tables [OBSERVED: McMillan, "Structured Context Engineering for File-Native Agentic Systems," arXiv:2602.05447, February 2026].

The findings challenge several widely held assumptions about context engineering. They also reveal a phenomenon the paper calls the "grep tax" -- and its implications extend well beyond file format selection.

## What the paper tested

The experimental setup measured how well LLM agents could navigate and retrieve information from large structured datasets. The key variables were:

- **File format:** YAML, Markdown, JSON, and TOON (Token-Oriented Object Notation, a compact format designed for token efficiency)
- **Architecture:** Whether the agent consumed data in-context or via file-based retrieval
- **Model tier:** Frontier models (Claude Opus 4.5, GPT-5.2, Gemini 2.5 Pro) vs. open-source alternatives
- **Scale:** Schemas with 10, 100, 1,000, and 10,000 tables

The methodology is rigorous: controlled experiments with statistical testing, not anecdotal benchmarking.

## Finding 1: Format does not drive accuracy

This is the headline result that will surprise many practitioners. Across all models and architectures, the choice of file format -- YAML vs. Markdown vs. JSON vs. TOON -- showed no statistically significant aggregate effect on accuracy (chi-squared = 2.45, p = 0.484) [OBSERVED: ibid.].

This does not mean format never matters. Individual models showed format-specific sensitivities -- some models performed measurably better with certain formats. But the effect was model-specific, not universal. There is no "best format for AI agents" that applies across the board.

The practical implication: if you are spending engineering time converting your data into a specific format because you believe it will improve agent accuracy, the evidence suggests that time is, in many setups, better spent elsewhere. Use whatever format your existing tooling already supports.

## Finding 2: Model capability dominates everything else

The most striking number in the paper is the gap between frontier and open-source model tiers: 21 percentage points in accuracy, "dwarfing any format or architecture effect" [OBSERVED: ibid.].

To put this in perspective: optimizing your file format, your chunking strategy, your retrieval architecture, and your prompt template -- all of these combined produce a smaller effect than simply using a more capable model.

This is both obvious and underappreciated. The AI tooling ecosystem spends enormous effort on retrieval optimization, prompt engineering, and context management. These matter. But the paper suggests they matter less than the underlying model capability -- at least for structured data navigation tasks.

*Trade-off:* Frontier models are expensive. The cost difference between running Opus and running an open-source model can be 10-50x per token. For high-volume production workloads, this cost difference is the constraint, not accuracy. The right choice depends on whether your bottleneck is quality or cost -- and that varies by use case [COMPARATIVE: frontier vs. open-source; conditions: structured data navigation tasks as defined in the paper; may not generalize to other task types].

## Finding 3: The grep tax

This is the paper's most novel contribution and the one with the broadest implications.

The "grep tax" is the observation that compact file formats -- formats designed to minimize token count -- can paradoxically consume significantly more tokens at scale. The mechanism: when an agent encounters an unfamiliar or highly compressed format, it compensates by issuing more search operations (greps, finds, reads) to build the context it needs to answer questions.

In other words, the tokens you save on the input are spent (and then some) on the agent's navigation behavior. File size does not predict runtime efficiency.

This finding has implications far beyond file format choice. It suggests a general principle about context engineering:

**Compressing context to save tokens can increase total token consumption by forcing the agent to reconstruct the context through repeated retrieval operations.**

This is the grep tax: the cost of not giving the agent enough context up front is that it spends tokens acquiring that context through trial and error.

## What this means for agent architecture

The grep tax is not specific to file formats. It applies to any system where an agent must navigate a large knowledge base with incomplete upfront context. Consider these common patterns:

**Minimal system prompts.** Some teams compress system prompts to save tokens, providing only the bare minimum context. The grep tax predicts that agents will compensate by asking more clarifying questions, making more tool calls, or generating exploratory code -- all of which consume tokens and increase latency.

**Aggressive context eviction.** When a context window fills up, many systems evict older content to make room for new content. If the evicted content is later needed, the agent must re-retrieve it -- paying the grep tax for content it already had.

**Retrieval-only memory.** RAG systems that provide small retrieved chunks without surrounding context force the agent to issue follow-up retrievals to understand the chunk in context. Each follow-up retrieval is a grep tax payment.

The general pattern: **any context management strategy that prioritizes token efficiency over context completeness will pay the grep tax in downstream token consumption.** The question is whether the net effect is positive or negative -- and the paper suggests it is often negative at scale.

## Finding 4: File-native agents scale to 10,000 tables

A more optimistic finding: frontier models using file-based retrieval successfully scaled to 10,000-table schemas using domain-partitioned file structures, while maintaining high navigation accuracy [OBSERVED: ibid.].

This matters because it establishes an upper bound that is higher than many practitioners assumed. The conventional wisdom has been that LLM agents struggle with large codebases and complex schemas. The paper suggests that with proper partitioning, frontier models can handle substantially larger structured datasets than previously demonstrated.

The key qualifier is "domain-partitioned." The scaling worked because the schemas were organized into logical partitions that the agent could navigate hierarchically. Dumping 10,000 table definitions into a single file would not produce the same result. Structure matters -- not the format of the structure, but the fact that structure exists.

## Finding 5: Architecture effects are model-dependent

File-based context retrieval improved accuracy for frontier models by +2.7% (p = 0.029) but degraded accuracy for open-source models by -7.7% (p < 0.001) [OBSERVED: ibid.].

This is a critical finding for teams choosing between in-context and retrieval-augmented architectures. The same architectural decision can help or hurt depending on the model being used. There is no universal answer to "should I use RAG or in-context?"

The practical implication: if you switch models, re-evaluate your architecture. What worked with GPT-5.2 may not work with Llama, and vice versa. Benchmark on your own data with your own model -- generic benchmarks will mislead you.

## The larger lesson: context quality over context quantity

Across all of the paper's findings, a pattern emerges: the quantity of context matters less than the quality and structure of context. Compact formats do not reliably help. Larger windows do not reliably help. What helps is:

1. **Using the most capable model you can afford** (21-point gap)
2. **Structuring context hierarchically** (enables scaling to 10,000 tables)
3. **Avoiding aggressive compression** (prevents the grep tax)
4. **Matching architecture to model tier** (architecture effects are model-dependent)

This is consistent with what practitioners building agent systems have observed empirically, but the paper provides the statistical rigor that anecdotal evidence lacks.

## How we think about this at ekkOS_

Our context management approach was designed around the principle that structured, pre-distilled context outperforms raw retrieval -- and this paper provides evidence for why. Rather than stuffing conversation history into the window and hoping the model finds what it needs, we maintain layered memory that surfaces distilled patterns, decisions, and outcomes. The grep tax is real, and we have measured it in our own system: agents that receive well-structured context up front issue fewer tool calls and produce higher-quality output per token spent. The constraint is that pre-distillation requires either human curation or a secondary model pass, both of which add latency. If you are designing a context management system, the paper's key insight applies directly: optimize for context quality and structure, not for raw token count.

## Practical takeaways

For teams building or evaluating AI agent systems, the paper suggests the following priorities:

1. **Stop optimizing file format.** Unless you have model-specific evidence that a particular format helps, use whatever your tooling already supports. The accuracy difference is not statistically significant.

2. **Invest in model capability first.** The 21-point gap between model tiers is the largest effect in the study. If you are running a less capable model to save money, measure whether the quality loss is costing you more in rework than the model savings.

3. **Watch for the grep tax.** If your agents are issuing many sequential tool calls or search operations, ask whether giving them more context up front would reduce total token consumption. The cheapest token is the one you do not have to retrieve twice.

4. **Partition your data structures.** The scaling results depend on hierarchical organization. If your agents are navigating large codebases or schemas, invest in logical partitioning rather than brute-force context stuffing.

5. **Benchmark with your model.** Architecture effects are model-dependent. Do not assume that what works for a frontier model will work for an open-source alternative, or vice versa.

The paper is available at [arXiv:2602.05447](https://arxiv.org/abs/2602.05447). At 9,649 experiments across 11 models, it is one of the most thorough empirical studies of agent context engineering published to date. It deserves close reading by anyone building systems where AI agents must navigate structured data -- which, increasingly, is most of us.
