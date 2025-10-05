---
title: quantization
tags: [Quantization, LLM, Optimization]
excerpt: 大语言模型量化技术综述，包括SmoothQuant、AWQ、LLM.int8、GPTQ、ZeroQuant、LUT-GEMM、SparseGPT等先进量化方法，以及weight-only量化在推理优化中的应用。
---

## LLM量化
https://zhuanlan.zhihu.com/p/616969812
- [SmoothQuant](https://github.com/mit-han-lab/smoothquant)
- Outlier Suppression
   - [Outlier Suppression](https://arxiv.org/abs/2209.13325)
   - [Outlier Suppression+](https://arxiv.org/abs/2304.09145)
- [AWQ](https://arxiv.org/abs/2306.00978)
 基于激活与参数大小的缩放保护
- [LLM.int8](https://arxiv.org/abs/2208.07339)
   - LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale
   - https://timdettmers.com/2022/08/17/llm-int8-and-emergent-features/
   - https://github.com/timdettmers/bitsandbytes
- [GPTQ](https://arxiv.org/pdf/2210.17323.pdf)
   - [GPTQ: 模型量化，穷鬼救星](https://zhuanlan.zhihu.com/p/616969812)
      不使用贪心算法，对W进行优化时，固定位置挑选Q
      W不同列间权重更新互相独立，使用批处理更新（Lazy Batch更新，group_size参数控制）
      数值稳定性优化
   - [4-bit LLM Quantization with GPTQ](https://mlabonne.github.io/blog/posts/4_bit_Quantization_with_GPTQ.html)
- [ZeroQuant](https://arxiv.org/abs/2206.01861)
   - https://github.com/microsoft/DeepSpeed/pull/2217
   - [ZeroQuant-V2: Exploring Post-training Quantization in LLMs from Comprehensive Study to Low Rank Compensation](https://arxiv.org/abs/2303.08302)
- [LUT-GEMM](https://arxiv.org/pdf/2206.09557.pdf)
   - [LUT-GEMM: 基于lut的量化矩阵乘法在大规模生成语言模型中的高效推理](https://blog.csdn.net/weixin_42764932/article/details/131230429?spm=1001.2014.3001.5501)
- [SparseGPT](https://arxiv.org/pdf/2301.00774.pdf)
- weight only
   - [Who Says Elephants Can’t Run: Bringing Large Scale MoE Models into Cloud Scale Production](https://arxiv.org/pdf/2211.10017.pdf) 
   - [TVM weight-only](https://github.com/apache/tvm/pull/15111)
   - [cutlass_fpA_intB_gemm](https://github.com/tlc-pack/cutlass_fpA_intB_gemm/pull/1/files)
