---
title: cutlass conv
tags: [CUTLASS, Convolution, CUDA]
excerpt: CUTLASS convolution implementation explained, including convolution parameter definitions (K, C, R, S), Conv2dProblemSize configuration, output size calculation formulas, and CUTLASS library applications in convolution operations.
---

## Convolution params: K, C, R, S
- R, S: height, width of the filter
- C number of input channels
- K filters, each produces a single output channel (plane)

```
cutlass::conv::Conv2dProblemSize(
    {1, 23, 56, 98},     // input size (NHWC)
    {128, 3, 3, 98},     // filter size (KRSC)
    {4, 0, 5, 0},      // padding (pad_h, _, pad_w, _)
    {3, 3},            // stride (stride_h, stride_w)
    {1, 1}             // dilation (dilation_h, dilation_w)
  )

  Conv2dProblemSize(
    cutlass::Tensor4DCoord input_size,    // NHWC
    cutlass::Tensor4DCoord filter_size,   // KRSC
    cutlass::Tensor4DCoord output_size,   // NPQK
    cutlass::conv::Mode mode = cutlass::conv::Mode::kCrossCorrelation,
    int split_k_slices = 1,
    int groups = 1
  );
  // set output P and Q
  P = ((H + pad_h * 2 - R * dilation_h) / stride_h) + 1;
  Q = ((W + pad_w * 2 - S * dilation_w) / stride_w) + 1;
```