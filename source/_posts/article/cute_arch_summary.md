---
title: CUTLASS Cute Arch 架构、指令、精度总结表
date: 2025-10-31
categories:
  - CUDA
  - CUTLASS
tags:
  - Tensor Core
  - 架构对照
  - 精度支持
  - 矩阵乘法
abbrlink: cute-arch-summary
description: CUDA各代Tensor Core（SM架构）对应CUTLASS Cute支持的MMA指令、尺寸和精度对照表，帮助开发者理解GPU架构演进与精度特性。
---

[TOC]

## 1 矩阵乘法加速器 (MMA) 架构、指令、精度对照表

| 架构 | 代号 | 指令类型 | MMA尺寸 | 输入精度A×B | 累加精度C | 输出精度D | 布局 | 特殊功能 |
|------|------|----------|---------|-------------|-----------|-----------|------|----------|
| **SM61** | Pascal | `dp4a.s32.s32` | - | U8×U8 | S32 | S32 | - | 点积操作 |
| **SM61** | Pascal | `dp2a.s32.s32` | - | U16×U8 | S32 | S32 | - | 点积操作 |
| **SM70** | Volta | `mma.sync` | 8×8×4 | F16×F16 | F16 | F16 | TN/NT/NN/TT | 首个Tensor Core |
| **SM75** | Turing | `mma.sync` | 16×8×8 | F16×F16 | F32 | F32 | TN | Tensor Core改进 |
| **SM75** | Turing | `mma.sync` | 8×8×16 | S8×S8 | S32 | S32 | TN | INT8支持 |
| **SM80** | Ampere | `mma.sync` | 16×8×8 | F16×F16 | F16/F32 | F16/F32 | TN/NT | 多种尺寸 |
| **SM80** | Ampere | `mma.sync` | 16×8×16 | F16×F16 | F16/F32 | F16/F32 | TN/NT | 多种尺寸 |
| **SM80** | Ampere | `mma.sync` | 16×8×8 | BF16×BF16 | F32 | F32 | TN/NT | BF16支持 |
| **SM80** | Ampere | `mma.sync` | 16×8×16 | BF16×BF16 | F32 | F32 | TN/NT | BF16支持 |
| **SM80** | Ampere | `mma.sync` | 16×8×32 | TF32×TF32 | F32 | F32 | TN/NT | TF32支持 |
| **SM80** | Ampere | `mma.sync` | 16×8×16 | S8×S8 | S32 | S32 | TN/NT | INT8 |
| **SM80** | Ampere | `mma.sync` | 16×8×32 | S8×U8/S8×S8 | S32 | S32 | TN/NT | INT8变体 |
| **SM80** | Ampere | `mma.sync` | 16×8×8 | S4×S4 | S32 | S32 | TN | INT4支持 |
| **SM80** | Ampere | `mma.sync` | 16×8×32 | S4×U4 | S32 | S32 | TN | INT4变体 |
| **SM89** | Ada Lovelace | `mma.sync` | 16×8×32 | E4M3×E4M3 | F32 | F32 | TN | FP8 (E4M3) |
| **SM89** | Ada Lovelace | `mma.sync` | 16×8×32 | E5M2×E5M2 | F32 | F32 | TN | FP8 (E5M2) |
| **SM89** | Ada Lovelace | `mma.sync` | 16×8×32 | E4M3×E5M2 | F32 | F32 | TN | FP8混合 |
| **SM89** | Ada Lovelace | `mma.sync` | 16×8×32 | E4M3×E4M3 | F16 | F16 | TN | FP8→F16 |
| **SM89** | Ada Lovelace | `mma.sync` | 16×8×32 | E5M2×E5M2 | F16 | F16 | TN | FP8→F16 |
| **SM90** | Hopper | `mma.sync` | 16×8×4 | F64×F64 | F64 | F64 | TN | 双精度支持 |
| **SM90** | Hopper | `mma.sync` | 16×8×8 | F64×F64 | F64 | F64 | TN | 双精度 |
| **SM90** | Hopper | `mma.sync` | 16×8×16 | F64×F64 | F64 | F64 | TN | 双精度 |
| **SM90** | Hopper | `wgmma.mma_async` | 64×N×16 | F16×F16 | F16/F32 | F16/F32 | SS/RS | 大型GMMA |
| **SM90** | Hopper | `wgmma.mma_async` | 64×N×16 | BF16×BF16 | F32 | F32 | SS/RS | 大型GMMA |
| **SM90** | Hopper | `wgmma.mma_async` | 64×N×8 | TF32×TF32 | F32 | F32 | SS/RS/TN | 大型GMMA |
| **SM90** | Hopper | `wgmma.mma_async` | 64×N×32 | S8×S8 | S32 | S32 | SS/RS/TN | 大型GMMA |
| **SM90** | Hopper | `wgmma.mma_async.sp` | 64×N×32 | F16×F16 | F16/F32 | F16/F32 | SS/RS | 稀疏GMMA |
| **SM90** | Hopper | `wgmma.mma_async.sp` | 64×N×32 | BF16×BF16 | F32 | F32 | SS/RS | 稀疏GMMA |
| **SM100** | Blackwell | `fma(float2)` | 2×1×1 | F32×F32 | F32 | F32 | - | float2数学 |
| **SM100** | Blackwell | `fma(float2)` | 1×2×1 | F32×F32 | F32 | F32 | - | float2数学 |
| **SM100** | Blackwell | UMMA | 64×N×8 | TF32*(TF32) | F32 | F32 | SS | UMMA操作 |
| **SM100** | Blackwell | UMMA | 64×N×16 | F16×F16 | F32 | F32 | SS | UMMA操作 |
| **SM100** | Blackwell | UMMA | 128×N×8 | TF32×TF32 | F32 | F32 | SS | UMMA操作 |
| **SM120** | 最新 | `mma.sync` | 16×8×32 | E2M1×E2M1 | F32 | F32 | TN | F6 (E2M1) |
| **SM120** | 最新 | `mma.sync` | 16×8×32 | E2M1×E3M2 | F32 | F32 | TN | F6混合 |
| **SM120** | 最新 | `mma.sync` | 16×8×32 | E2M1×E2M3 | F32 | F32 | TN | F6/F4混合 |
| **SM120** | 最新 | `mma.sync` | 16×8×32 | E2M1×E4M3 | F32 | F32 | TN | F6/F8混合 |
| **SM120** | 最新 | `mma.sync` | 16×8×32 | E3M2 REFERENCES | F32 | F32 | TN | F6变体 |
| **SM120** | 最新 | `mma.sync` | 16×8×32 | E4M3×E2M1 | F32 | F32 | TN | F8/F6混合 |
| **SM120** | 最新 | `mma.sync` | 16×8×32 | E5M2 REFERENCES | F32 | F32 | TN | F6变体 |

**说明：**
- **布局**：TN=转置×非转置, NT=非转置×转置, NN=非转置×非转置, TT=转置×转置, SS=共享内存, RS=寄存器
- **精度缩写**：F16=FP16, F32=FP32, F64=FP64, BF16=Bfloat16, TF32=TF32, S8/U8=INT8, S4/U4=INT4
- **E4M3/E5M2**：FP8格式 (4位指数+3位尾数 / 5位指数+2位尾数)
- **E2M1/E3M2/E2M3**：FP6/FP4格式

## 2 内存拷贝操作 (Copy) 架构、指令、精度对照表

| 架构 | 代号 | 指令类型 | 操作类型 | 数据类型 | 缓存级别 | 特殊功能 |
|------|------|----------|----------|----------|----------|----------|
| **SM50** | Maxwell | `shfl.sync` | Shuffle | U32 | - | Warp内数据交换 |
| **SM75** | Turing | `ldmatrix.sync` | LDSM | U16/U32 | Shared | 共享内存矩阵加载 |
| **SM75** | Turing | `movmatrix.sync` | MOVM | U32 | Register | 寄存器矩阵转置 |
| **SM80** | Ampere | `cp.async` | Async Copy | 多种 | Shared | 异步拷贝 |
| **SM90** | Hopper | `cp.async.bulk.tensor` | TMA | 多种 | Shared/L2 | 张量内存加速器 |
| **SM90** | Hopper | `cp.async.bulk.prefetch.tensor` | TMA Prefetch | 多种 | L2 | TMA预取 |
| **SM100** | Blackwell | `ld.global.L1::no_allocate.v8.f32` | Load 256bit | F32 | L1 | 256bit加载 |
| **SM100** | Blackwell | `st.global.L1::no_allocate.v这是因为8.f32` | Store 256bit | F32 | L1 | 256bit存储 |
| **SM100** | Blackwell | `ldsm.sync` | LDSM | U8/U16/U32 | Shared | 共享内存加载 |
| **SM100** | Blackwell | `stsm.sync` | STSM | U8/U16/U32 | Shared | 共享内存存储 |
| **SM100** | Blackwell | `cp.async.bulk.tensor` берег | TMA | 多种 | Shared/L2 | 优化的TMA |

**说明：**
- **LDSM**：Load Matrix (从共享内存加载矩阵到寄存器)
- **STSM**：Store Matrix (从寄存器存储矩阵到共享内存)
- **TMA**：Tensor Memory Accelerator (张量内存加速器)
- **MOVM**：Move Matrix (矩阵数据移动和转置)

## 3 完整精度支持汇总

### 支持的数值类型
1. **浮点精度**：F16, BF16, TF32, F32, F64
2. **FP8格式**：E4M3, E5M2
3. **FP6/F4格式** (SM120)：E2M1, E3M2, E2M3
4. **整数精度**：S8, U8, S4, U4
5. **复数**：C64 (complex double)
6. **混合精度**：F16→F32, BF16→F32, TF32→F32, FP8→F32/F16

### 架构演进特点
- **SM61-SM75**：基础MMA和拷贝操作
- **SM80**：大幅改进，支持多种精度和尺寸
- **SM89**：引入FP8支持
- **SM90**：GMMA大型操作和稀疏矩阵支持
- **SM100**：float2数学和UMMA操作
- **SM120**：FP6/F4混合精度支持

