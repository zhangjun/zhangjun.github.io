---
title: XAttention - Block Sparse Attention with Antidiagonal Scoring
tags: [Paper Summary, Attention, Optimization]
excerpt: Implements efficient block sparse attention mechanism through antidiagonal scoring, achieving 13.5x speedup in long-context Transformer models while maintaining high accuracy across natural language, video understanding, and video generation domains.
---

# Summary of "XAttention: Block Sparse Attention with Antidiagonal Scoring"

## Key Contributions

This paper introduces XAttention, a novel framework that significantly accelerates long-context inference in Transformer models by implementing an efficient block-sparse attention mechanism. The key innovation is using the sum of antidiagonal values in the attention matrix as a lightweight yet effective proxy for block importance.

## Core Problem Addressed

Long-Context Transformer Models (LCTMs) suffer from the quadratic computational complexity of attention mechanisms, creating bottlenecks during the pre-filling stage. While block-sparse attention methods exist, they struggle with balancing accuracy and efficiency due to costly block importance measurements.

## Methodology

XAttention employs a three-step process:
1. **Antidiagonal Scoring**: Sums values along strided antidiagonals within each attention block to determine importance
2. **Threshold Block Selection**: Selects high-scoring blocks based on a predefined threshold
3. **Minimum Threshold Prediction**: Uses dynamic programming to determine optimal thresholds for each attention head

The antidiagonal pattern effectively intersects both vertical and slash patterns within blocks, enabling efficient detection of important attention patterns without the computational overhead of existing methods.

## Results

XAttention was evaluated across multiple domains:
- **Natural Language**: Outperformed baseline methods on RULER and LongBench benchmarks
- **Video Understanding**: Achieved the best average scores on VideoMME, even outperforming full attention on long videos
- **Video Generation**: Maintained high fidelity compared to full attention on VBench with over 50% sparsity

Performance highlights include:
- Up to 13.5× acceleration in attention computation
- Maintained comparable accuracy to full attention
- Pattern selection up to 24.9× faster than competing methods

## Significance

XAttention provides a plug-and-play solution for accelerating long-context Transformer models without requiring retraining. By dramatically reducing computational costs while preserving model accuracy, it enables more efficient deployment of LCTMs for real-world applications, particularly in multimodal domains like video understanding and generation.