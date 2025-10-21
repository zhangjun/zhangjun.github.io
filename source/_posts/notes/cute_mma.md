---
title: cute_mma.md
date: 2025-10-22 00:38:22
tags: [cutlass, cute]
excerpt: intro for cute mma
---

## MMA
```cpp
struct SM80_16x8x8_F32F16F16F32_TN
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
    asm volatile(
      "mma.sync.aligned.m16n8k8.row.col.f32.f16.f16.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
  }
};
```
### MMA Operation
- Operation 结构体名称

  [gpu arch]\_[MNK dimensions]\_[types]\_[arrangement of the A and B inputs]

  - SM80 指 Ampere GPU 架构名称
  - 16x8x8 分别代表 MMA 操作中 M、N、K 维度，对应 ptx 指令 .m16n8k8.
  - F32F16F16F32 分别指四个矩阵操作数的元素类型。MMA 用于计算 D=A*B+C, 对应数据类型从左到右读取(D-F32, A-F16, B-F16, C-F32). 对应 ptx 指令名称为 .f32.f16.f16.f32
  - NT 代表 A 矩阵 column major(M-major), B 矩阵 row major(N-major), 对应 ptx 指令为 .col.row.

### MMA_Traits