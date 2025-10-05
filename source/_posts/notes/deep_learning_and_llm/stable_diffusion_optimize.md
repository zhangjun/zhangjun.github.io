---
title: stable diffusion optimization
tags: [Stable Diffusion, Optimization, PaddlePaddle]
excerpt: Stable Diffusion推理优化技术详解，包括Flash Attention、Norm融合、混合Layout计算、推理显存优化等核心技术，实现512×512图像0.76秒生成，性能超越TensorRT 7.9%。
---

## stable diffusion
- norm+act
- add_bias_norm
- add_norm_act
- add_bias_add_norm_act

## paddle stable diffusion optimization

基于PaddlePaddle对Stable Diffusion进行推理时，512*512图像生成速度68.2 iters/s，实现 0.76s 出图。其推理速度是 Diffusers（PyTorch）的4倍，比TensorRT最优速度快7.9%，同时显存占用仅为TensorRT的43%。

- Flash Attention

飞桨一直致力于大模型推理优化，支持多种通用Transformer类结构的高性能推理优化。在Stable Diffusion模型推理中，飞桨集成的高性能的Flash Attention kernel，通过将attention中的softmax计算进行拆解、分片计算，大量减少推理过程中self-attention和cross-attention计算对显存的访问次数，同时实现了推理加速和显存优化。

- Norm融合

Norm是Stable Diffusion中U-Net常用算子，主要分为LayerNorm和GroupNorm。LayerNorm和GroupNorm算子作为批规约运算，能够很好地和前后的elementwise类型、激活类型算子进行融合，消除算子间的显存访问。飞桨对LayerNorm和GroupNorm与前后算子的4种不同pattern进行了融合，共融合了93个Norm结构，提升了3%的推理性能。
![image](https://github.com/zhangjun/zhangjun.github.io/assets/1312389/45db9b78-cdc4-4f0d-b098-93a4e7b2c40f)

- 混合Layout计算

通过对模型张量排布匹配优化，支持不同的Layout消除和合并U-Net中的转置操作，提高了推理速度同时也能降低了运行显存占用，共减少了32次转置操作，带来了3~4%的推理性能提升。整体显存占用降低约19%
![image](https://github.com/zhangjun/zhangjun.github.io/assets/1312389/40b74c4d-7c6e-44d4-9a17-5741aea70632)

- 推理显存优化

推理workspace复用技术
