---
title: Prompt Cache - Modular Attention Reuse for Low-Latency Inference
tags: [Paper Summary, LLM, Optimization]
excerpt: Accelerates large language model inference through modular attention reuse technology, achieving 8x GPU inference and 60x CPU inference TTFT latency improvements, particularly suitable for long-context applications like document QA and recommendation systems.
---

# Summary of "Prompt Cache: Modular Attention Reuse for Low-Latency Inference"

This paper introduces Prompt Cache, a novel approach for accelerating large language model (LLM) inference by reusing attention states across different prompts. The key contributions include:

## Core Insight and Approach
- Many LLM prompts contain overlapping text segments (system messages, templates, documents)
- Prompt Cache precomputes and stores attention states for these frequently occurring segments
- When these segments appear in user prompts, the cached states are reused instead of recomputing them

## Technical Implementation
- Introduces a Prompt Markup Language (PML) to make reusable text segments explicit as "prompt modules"
- Employs a schema to define these modules and ensure positional accuracy during attention state reuse
- Leverages the finding that LLMs can operate on attention states with discontinuous position IDs
- Extends the traditional Key-Value (KV) Cache from single-prompt reuse to cross-prompt reuse

## Performance Improvements
- Significantly reduces time-to-first-token (TTFT) latency:
  - 8× improvement for GPU-based inference
  - 60× improvement for CPU-based inference
- Maintains output accuracy without requiring model parameter modifications
- Benefits increase with prompt length and model size (quadratic improvement)

## Applications
- Particularly effective for long-context applications like document-based QA and recommendations
- Demonstrated use cases include code generation, personalization, and parameterized prompts

## Implementation Details
- Built on HuggingFace transformers library
- Compatible with various Transformer architectures (Llama2, Falcon, MPT)
- Can store prompt modules in either CPU or GPU memory, with different trade-offs

The paper presents a practical approach to LLM inference optimization that addresses the computational bottleneck of processing repetitive prompt segments, with significant latency improvements especially for longer prompts.