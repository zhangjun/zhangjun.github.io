---
title: stable diffusion optimization
tags: [Stable Diffusion, Optimization, PaddlePaddle]
excerpt: Stable Diffusion推理优化技术详解，包括Flash Attention、Norm融合、混合Layout计算、推理显存优化等核心技术，实现512×512图像0.76秒生成，性能超越TensorRT 7.9%。
---

## 一、Stable Diffusion 算子融合优化

- `norm + act`
- `add_bias + norm`
- `add + norm + act`
- `add_bias + add + norm + act`

## 二、Paddle Stable Diffusion 推理优化

基于 PaddlePaddle 对 Stable Diffusion 进行推理时，`512x512` 图像生成速度达到 `68.2 iters/s`，实现 `0.76 s` 出图。其推理速度是 Diffusers（PyTorch）的 4 倍，比 TensorRT 最优速度快 7.9%，同时显存占用仅为 TensorRT 的 43%。

### 1. Flash Attention

飞桨一直致力于大模型推理优化，支持多种通用 Transformer 类结构的高性能推理优化。在 Stable Diffusion 模型推理中，飞桨集成了高性能 Flash Attention kernel，通过将 attention 中的 softmax 计算进行拆解、分片计算，大量减少推理过程中 self-attention 和 cross-attention 对显存的访问次数，同时实现推理加速与显存优化。

### 2. Norm 融合

Norm 是 Stable Diffusion 中 U-Net 常用算子，主要分为 LayerNorm 和 GroupNorm。LayerNorm 和 GroupNorm 作为批规约运算，能够很好地和前后 elementwise 类型、激活类型算子进行融合，消除算子间的显存访问。飞桨对 LayerNorm 和 GroupNorm 与前后算子的 4 种不同 pattern 进行了融合，共融合了 93 个 Norm 结构，提升了 3% 的推理性能。
![image](https://github.com/zhangjun/zhangjun.github.io/assets/1312389/45db9b78-cdc4-4f0d-b098-93a4e7b2c40f)

### 3. 混合 Layout 计算

通过对模型张量排布匹配优化，支持不同 Layout，消除并合并 U-Net 中的转置操作，提高推理速度的同时降低运行显存占用。该优化共减少了 32 次转置操作，带来 3%~4% 的推理性能提升，整体显存占用降低约 19%。
![image](https://github.com/zhangjun/zhangjun.github.io/assets/1312389/40b74c4d-7c6e-44d4-9a17-5741aea70632)

### 4. 推理显存优化

推理 workspace 复用技术。
