---
title: performance optimization
tags: [Performance, Optimization, Deep Learning]
excerpt: 性能优化技术总结，涵盖CPU和GPU的理论峰值计算性能、内存带宽计算方法，以及深度学习模型优化策略，包括算子优化、图优化、算子融合等技术。
---


## [Performance Measurement](https://www.sciencedirect.com/science/article/pii/B978012416970800002X)

### CPU

* theoretical peak

   two Intel Xeon E5-2697 v2 (2S-E5) with 12 cores per CPU, each running at 2.7 GHz without turbo mode. These processors support the AVX extension with 256-bit SIMD instructions that can process 8 single precision (32 bits) numbers per CPU cycle.

   > _theoretical peak Flop/s is 2.7 (GHz) × 8 (SP FP) × 2 (ADD/MULL) × 12 (cores) × 2 (CPUs) = 1036.8 GFlop/s._

* memory bandwidth

   theoretical [memory bandwidth](https://www.sciencedirect.com/topics/computer-science/memory-bandwidth) is computed from the [memory frequency](https://www.sciencedirect.com/topics/computer-science/memory-frequency) (1866 GHz), the number of channels (4), the number of bytes transferred by channel per cycle (8), which gives 1866 × 4 × 8 × 2 (# of processors) = 119 GByte/s [peak bandwidth](https://www.sciencedirect.com/topics/computer-science/peak-bandwidth) for the dual socket 2S-E5 system.

### GPU
* theoretical peak

   Theoretical peak FLOPS can be calculated as follows:

   > FLOPS = Number of CUDA Cores × Clock Speed (GHz) × Operations per Cycle

   For example, an NVIDIA Tesla V100 GPU has 5120 CUDA cores and a clock speed of 1.53 GHz. Assuming it can perform 2 floating-point operations per cycle, the theoretical peak FLOPS would be:
   FLOPS = 5120 × 1.53 × 2 ≈ 15.7 TFLOPS

* memory bandwidth

   Theoretical memory bandwidth can be calculated using the formula:

   > Memory Bandwidth = Memory Clock Speed (GHz) × Memory Bus Width (bits) / 8 (to convert bits to bytes) × 2 (for DDR)

   For instance, if an NVIDIA Tesla V100 has a memory clock speed of 1.75 GHz and a memory bus width of 4096 bits, the theoretical memory bandwidth would be:

   > Memory Bandwidth = 1.75 × 4096 / 8 × 2 ≈ 900 GB/s

* [GPU Architecture: A Look at Peak FLOPS and Memory Bandwidth](https://developer.nvidia.com/blog/gpu-architecture-a-look-at-peak-flops-and-memory-bandwidth/)

## 算子性能优化

### Conv
* Winograd

   3x3s1

* DepthWise

   groups = ic = oc

* DirectConv

   3x3s2

* Img2Col+GEMM

   https://www.zhihu.com/question/68435920/answer/1770994396


## 图优化
### Layout 调优
* LayoutSensitiveOps
   ```shell
   norm
   conv
   pool
   resize
   fused_conv
   grid_simple
   ```

## 优化
计算时间由数据移动效率和计算效率决定
- 数据移动效率：CPU/GPU内存读写、PCIe数据传输、GPU kernel launch
- 计算效率：计算单元速度

提升计算密度，即在模型FLOPs相当的情况下减少访存量，减少kernel个数，提高FLOPs/byte或FLOPs/kernel_launch。例如：尽量减少使用tensor变换算子（Concat/Split/Transpose等算子）、减少Tile/Gather等算子导致的中间结果内存读写膨胀、尽量增大每次请求的batch size等。

### 计算优化
### 访存优化

### 图优化
#### 无用节点消除
- Unsqueeze 常量输入
- 当Slice Op的index_start等于0、index_end等于c-1时该Op无意义可被删除
- Expand Op指定的输出shape等于输入shape时该Op无意义可被删除
- 连续的Reshape只需要保留最后一个reshape
- 连续的内存排布转换只需要保留最后一个
- Concat Split Elimination 合并后又进行同样的拆分，可同时删除这两个Op
#### 算子融合
- Matmul+Add->Gemm
- Conv+Add
- Conv+Act
- Conv+BN
- MatMul+Scale 融合到alpha
- Gemm+Act
- Gemm+elementwise
#### 算子变换
- BN->Scale 计算量更少
- Matmul -> Conv
- FC -> Conv1x1
- Matmul+Transpose
- ShuffleChannel -> Reshape+Transpose