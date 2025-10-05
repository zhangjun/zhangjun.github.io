---
title: gemm optimize
tags: [GEMM, GPU, Optimization]
excerpt: GEMM矩阵乘法优化技术详解，包括基础概念、向量内积和外积优化方法、双缓冲技术等核心优化策略，帮助提升GPU上矩阵运算性能。
---

## gemm basic
<img width="627" alt="image" src="https://user-images.githubusercontent.com/1312389/230854659-ef27dcf0-f449-4549-9c04-947ac79dd875.png">

## optimize
- 向量内积   
   ![image](https://github.com/zhangjun/zhangjun.github.io/assets/1312389/4a95f5b0-90b1-46ab-9ed5-e324086e8bee)
- 向量外积
   ![image](https://github.com/zhangjun/zhangjun.github.io/assets/1312389/61a36b83-a8be-46a2-909a-2153f99d79ef) 
- double buffer
