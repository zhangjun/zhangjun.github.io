---
title: tensor core
tags: [Tensor Core, GPU, NVIDIA]
excerpt: NVIDIA Tensor Core技术详解，包括第一代、第二代、第三代Tensor Core的架构特点、计算能力和性能指标，以及在不同GPU架构中的实现差异。
---

## Tensor Core
### 1st
4 * 2 * 64 FP16 FMA/clock = 512 per SM per clock
<img width="291" alt="image" src="https://user-images.githubusercontent.com/1312389/184502979-753e05dc-77eb-4c41-8358-7fbede365928.png">

### 2nd

### 3rd
4 * 1 * 256 FP16 FMA/clock = 1024 per SM per clock

<img width="901" alt="image" src="https://user-images.githubusercontent.com/1312389/184522109-7f9e789e-c858-4a6a-91a7-ded7786f1742.png">

![image](https://user-images.githubusercontent.com/1312389/188293448-70b5780d-a994-4e55-93dd-13e2c0896d49.png)
