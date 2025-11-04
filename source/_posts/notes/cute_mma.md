---
title: cute_mma.md
date: 2025-10-22 00:38:22
tags: [cutlass, cute]
excerpt: intro for cute mma
---

[TOC]

## 1 arch
### 1.1 mma
### 1.2 copy

## 2 MMA
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


// (T32,V1) -> (M8,N8)
using SM80_8x4      = Layout<Shape <Shape < _4,_8>,_1>,
                             Stride<Stride< _8,_1>,_0>>;
// (T32,V2) -> (M8,N8)
using SM80_8x8_Row  = Layout<Shape <Shape < _4,_8>,_2>,
                             Stride<Stride<_16,_1>,_8>>;
// (T32,V4) -> (M8,N16)
using SM80_8x16_Row = Layout<Shape <Shape < _4,_8>,_4>,
                             Stride<Stride<_32,_1>,_8>>;
// (T32,V4) -> (M16,N8)
using SM80_16x8_Row = Layout<Shape <Shape < _4,_8>,Shape < _2,_2>>,
                             Stride<Stride<_32,_1>,Stride<_16,_8>>>;

////////////////////////////////////////////
//////// fp16 = fp16 * fp16 + fp16 /////////
////////////////////////////////////////////
template <>
struct MMA_Traits<SM80_16x8x8_F16F16F16F16_TN>
{
  using ValTypeD = half_t;
  using ValTypeA = half_t;
  using ValTypeB = half_t;
  using ValTypeC = half_t;

  using Shape_MNK = Shape<_16,_8,_8>;
  using ThrID   = Layout<_32>;
  using ALayout = SM80_16x8_Row;
  using BLayout = SM80_8x8_Row;
  using CLayout = SM80_16x8_Row;
};

//////////////////////////////////////////
/////// fp32 = fp16 * fp16 + fp32 ////////
//////////////////////////////////////////
template <>
struct MMA_Traits<SM80_16x8x8_F32F16F16F32_TN>
     : MMA_Traits<SM80_16x8x8_F16F16F16F16_TN>
{
  using ValTypeD = float;
  using ValTypeA = half_t;
  using ValTypeB = half_t;
  using ValTypeC = float;
};

```
### 2.1 MMA Operation
- Operation 结构体名称

  [gpu arch]\_[MNK dimensions]\_[types]\_[arrangement of the A and B inputs]

  - SM80 指 Ampere GPU 架构名称
  - 16x8x8 分别代表 MMA 操作中 M、N、K 维度，对应 ptx 指令 .m16n8k8.
  - F32F16F16F32 分别指四个矩阵操作数的元素类型。MMA 用于计算 D=A*B+C, 对应数据类型从左到右读取(D-F32, A-F16, B-F16, C-F32). 对应 ptx 指令名称为 .f32.f16.f16.f32
  - NT 代表 A 矩阵 column major(M-major), B 矩阵 row major(N-major), 对应 ptx 指令为 .col.row.

### 2.2 MMA_Traits
```cpp
template <class MMAOperation, class... MMAOpArgs>
struct MMA_Traits
{
  static_assert(sizeof(MMAOperation) == 0, "MMA_Traits not implemented for this MMA_Operation.");
};

template <class D, class A, class B, class C>
struct MMA_Traits<UniversalFMA<D,A,B,C>>
{
  using ValTypeD = D;
  using ValTypeA = A;
  using ValTypeB = B;
  using ValTypeC = C;

  // Logical shape of the MMA
  using Shape_MNK = Shape<_1,_1,_1>;

  // Logical thread id (tid) -> tidx
  using ThrID   = Layout<_1>;

  // (Logical thread id (tid), Logical value id (vid)) -> coord

  // (tid,vid) -> (m,k)
  using ALayout = Layout<Shape<_1,_1>>;
  // (tid,vid) -> (n,k)
  using BLayout = Layout<Shape<_1,_1>>;
  // (tid,vid) -> (m,n)
  using CLayout = Layout<Shape<_1,_1>>;
};

// Extract an MMA_Op from an MMA_Traits
template <class MMA_Traits>
struct MMA_Op {};

template <class MMA_Op_Arg, class... Args>
struct MMA_Op<MMA_Traits<MMA_Op_Arg, Args...>> {
  using type = MMA_Op_Arg;
};
```
### 2.3 TiledMMA

## 3 Atom
### 3.1 MMA_Atom
```cpp
template <class... Args>
struct MMA_Atom;

template <class MMAOperation>
struct MMA_Atom<MMAOperation> : MMA_Atom<MMA_Traits<MMAOperation>>
{};

template <class MMAOperation, class... Args>
struct MMA_Atom<MMA_Traits<MMAOperation, Args...>>
  : MMA_Traits<MMAOperation, Args...>
{
  using MMA_Op = MMAOperation;
  using Traits = MMA_Traits<MMAOperation, Args...>;

  // Element value types from the MMA_Traits
  using ValTypeD = typename Traits::ValTypeD;
  using ValTypeA = typename Traits::ValTypeA;
  using ValTypeB = typename Traits::ValTypeB;
  using ValTypeC = typename Traits::ValTypeC;

  // Thr-Val layouts from the MMA_Traits
  using Shape_MNK  = typename Traits::Shape_MNK;
  using ThrID      = typename Traits::ThrID;
  using LayoutC_TV = typename Traits::CLayout;
  using LayoutA_TV = typename Traits::ALayout;
  using LayoutB_TV = typename Traits::BLayout;

  // Fragment value types from the MMA_Traits (optional, defaults to Val type)
  using FrgTypeD = typename detail::FrgTypeC_or_Default<Traits>::type;
  using FrgTypeA = typename detail::FrgTypeA_or_Default<Traits>::type;
  using FrgTypeB = typename detail::FrgTypeB_or_Default<Traits>::type;
  using FrgTypeC = typename detail::FrgTypeC_or_Default<Traits>::type;
};

template <class TiledMMA, class ThrCoord>
struct ThrMMA;

// @tparam MMA_Atom The MMA_Atom to use in the TiledMMA
// @tparam AtomLayoutMNK The MNK-tiling of the Atom to be performed.
// @tparam PermuationsMNK Permutations to apply to each MNK-mode before tiling for the Atom.
template <class MMA_Atom,
          class AtomLayoutMNK,
          class PermutationMNK = Tile<Underscore,Underscore,Underscore>>
struct TiledMMA : MMA_Atom
{
  using Atom           = MMA_Atom;
  using AtomShape_MNK  = typename MMA_Atom::Shape_MNK;
  using AtomThrID      = typename MMA_Atom::ThrID;
  using AtomLayoutC_TV = typename MMA_Atom::LayoutC_TV;
  using AtomLayoutA_TV = typename MMA_Atom::LayoutA_TV;
  using AtomLayoutB_TV = typename MMA_Atom::LayoutB_TV;

  static_assert(   rank_v<AtomLayoutMNK>  == 3,   "TiledMMA requires rank-3 AtomLayoutMNK");
  static_assert(   rank_v<PermutationMNK> == 3,   "TiledMMA requires rank-3 PermutationMNK");
  static_assert( is_tuple<PermutationMNK>::value, "TiledMMA requires independent permutations of MNK.");
  static_assert(is_static<PermutationMNK>::value, "TiledMMA requires static permutations of MNK.");

  using ThrLayoutVMNK = decltype(tiled_product(AtomThrID{}, AtomLayoutMNK{}));
  ThrLayoutVMNK thr_layout_vmnk_;

  ...
};

template <class TiledMMA, class ThrVMNK>
struct ThrMMA : TiledMMA
{
  ...
};
```

- make_tiled_mma

### 3.2 Copy_Atom
```cpp
template <class... Args>
struct Copy_Atom;

template <class CopyOperation, class CopyInternalType>
struct Copy_Atom<CopyOperation, CopyInternalType> : Copy_Atom<Copy_Traits<CopyOperation>, CopyInternalType>
{};

template <class... Args, class CopyInternalType>
struct Copy_Atom<Copy_Traits<Args...>, CopyInternalType>
  : Copy_Traits<Args...>
{
  ...
};

template <class TiledCopy, class ThrIdx>
struct ThrCopy;

template <class Copy_Atom,
          class LayoutCopy_TV,  // (tid,vid) -> coord   [Need not be 2D...]
          class ShapeTiler_MN>  // coord space
struct TiledCopy : Copy_Atom
{
  ...
};

template <class TiledCopy, class ThrIdx>
struct ThrCopy
{
  ...
};

template <class... Args,
          class LayoutCopy_TV,
          class Tiler>
CUTE_HOST_DEVICE
auto
make_tiled_copy_impl(Copy_Atom<Args...> const& atom,
                     LayoutCopy_TV      const&,
                     Tiler              const&)
{
  return TiledCopy<Copy_Atom<Args...>, LayoutCopy_TV, Tiler>{atom};
}
```

- make_tiled_copy