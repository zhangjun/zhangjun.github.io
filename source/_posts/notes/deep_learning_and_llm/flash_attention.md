---
title: flash attention
tags: [Flash Attention, Transformer, GPU Optimization]
excerpt: Flash Attention technology explained, including parallelization strategies, work partition optimization, supported head dimensions, and Flash Attention2's fused kernels, matrix tiling, causal masking, and other core optimization techniques.
---

# Flash Attention
## [Flash Attention](https://openreview.net/pdf?id=H4DqfPSibmx)
- parallelism
  parallelize over batch_size and num of heads
  flash attention2 - long sequences(small batch size or num of heads), parallelize over sequence length dimension
- better work partition
  reduce the amount of synchronization and communication between different warps
  FlashAttention splits K and V across 4 warps while keeping Q accessible by all warps.
  FlashAttention-2 splits Q across 4 warps while keeping K and V accessible by all warps.
  ![image](https://github.com/zhangjun/zhangjun.github.io/assets/1312389/703cdb9d-927b-4316-85ad-58d380b9478d)
- supported head dimensions up to 256 and MQA
   GQA、MQA
## Flash Attention2 优化
<img width="657" alt="image" src="https://github.com/zhangjun/zhangjun.github.io/assets/1312389/2cadba46-f1e4-4b4b-a2bc-aaa0c0061b4d">

Flash Attention2优化点详解
- Fused kernel与矩阵分块
- Causal Masking
- Non-Matmul 计算优化
- 流水编排与异步加载和Double Buffer
- Layout Swizzle