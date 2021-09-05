---
title: conv
date: 2021-04-10 11:04:13
tags:
---

conv详细介绍。

# conv2d
```
inline bool IsExpand(const std::vector<int64_t>& filter_dim,
                     const std::vector<int>& strides,
                     const std::vector<int>& paddings,
                     const std::vector<int>& dilations) {
  bool filter_1 = true, strides_1 = true, padding_0 = true, dilation_1 = true;
  for (size_t j = 0; j < strides.size(); ++j) {
    filter_1 = filter_1 && (static_cast<int>(filter_dim[j + 2]) == 1);
    strides_1 = strides_1 && (strides[j] == 1);
    padding_0 = padding_0 && (paddings[j] == 0);
    dilation_1 = dilation_1 && (dilations[j] == 1);
  }
  return !(filter_1 && strides_1 && padding_0 && dilation_1);
}
```
```
// use col_shape in the im2col calculation
// col_shape_vec:
// {i_c/g, k_h, k_w, o_h, o_w} or {i_c/g, k_d, k_h, k_w, o_d,o_h, o_w}
```
gemm calc
```
// use col_matrix_shape in the gemm calculation size:
// (i_c/g * k_h * k_w, o_h * o_w) or (i_c/g * k_d * k_h * k_w, o_d * o_h * o_w)
```

```
static inline size_t naive_conv_out_size(size_t in_size, size_t pad,
                                         size_t dilation, size_t ksize,
                                         size_t stride) {
    return (in_size + 2 * pad - dilation * (ksize - 1) - 1) / stride + 1;
}

static inline void naive_conv_fwd_nchw(const float *src, const float *filter,
                                       float *dst, size_t n, size_t w, size_t h,
                                       size_t c, size_t k, size_t fx, size_t fy,
                                       size_t px, size_t py, size_t sx,
                                       size_t sy, size_t dx, size_t dy, size_t group) {
    size_t oh = naive_conv_out_size(h, py, dy, fy, sy);
    size_t ow = naive_conv_out_size(w, px, dx, fx, sx);
    assert((group >= 1) && (c % group == 0) && (k % group == 0));
    size_t k_per_group = k / group;
    size_t c_per_group = c / group;
        size_t ig, in, ik, ioh, iow, ic, is, ir;
    size_t cur_h, cur_w, o_idx, i_idx, f_idx;
    // input:[n,c,h,w], filter:[k, c, fx, fy], output: [n, k, out_h, out_w]
    for (ig = 0; ig < group; ig++) {
        for (in = 0; in < n; in++) {
            for (ik = 0; ik < k_per_group; ik++) {
                for (ioh = 0; ioh < oh; ioh++) {
                    for (iow = 0; iow < ow; iow++) {
                        // sliding window for this filter
                        float value = .0f;
                        o_idx = in * k * oh * ow + ig * k_per_group * oh * ow + ik * oh * ow + ioh * ow + iow;
                        for (ic = 0; ic < c_per_group; ic++) {
                            for (ir = 0; ir < fy; ir++) {
                                cur_h = sy * ioh - py + dy * ir;
                                if (cur_h < 0 || cur_h >= h)
                                    continue;
                                for (is = 0; is < fx; is++) {
                                    cur_w = sx * iow - px + dx * is;
                                    if (cur_w < 0 || cur_w >= w)
                                        continue;
                                    i_idx = in * c * h * w + ig * c_per_group * h * w + ic * h * w +
                                            cur_h * w + cur_w;
                                    f_idx = ig * k_per_group * c_per_group * fy * fx + ik * c_per_group * fy * fx + ic * fy * fx +
                                            ir * fx + is;
                                    value += src[i_idx] * filter[f_idx];
                                }
                            }
                        }
                        dst[o_idx] = value;
                    }
                }
            }
        }
    }
}

// group = 1
static inline void naive_conv_fwd_nchw(const float *src, const float *filter,
                                       float *dst, size_t n, size_t w, size_t h,
                                       size_t c, size_t k, size_t fx, size_t fy,
                                       size_t px, size_t py, size_t sx,
                                       size_t sy, size_t dx, size_t dy, size_t group) {
    size_t oh = naive_conv_out_size(h, py, dy, fy, sy);
    size_t ow = naive_conv_out_size(w, px, dx, fx, sx);
    assert((group >= 1) && (c % group == 0) && (k % group == 0));
    size_t k_per_group = k / group;
    size_t c_per_group = c / group;
        size_t ig, in, ik, ioh, iow, ic, is, ir;
    size_t cur_h, cur_w, o_idx, i_idx, f_idx;
    // input:[n,c,h,w], filter:[k, c, fx, fy], output: [n, k, out_h, out_w]
    for (ig = 0; ig < group; ig++) {
        for (in = 0; in < n; in++) {
            for (ik = 0; ik < k_per_group; ik++) {
                for (ioh = 0; ioh < oh; ioh++) {
                    for (iow = 0; iow < ow; iow++) {
                        // sliding window for this filter
                        float value = .0f;
                        o_idx = in * k * oh * ow + ig * k_per_group * oh * ow + ik * oh * ow + ioh * ow + iow;
                        for (ic = 0; ic < c_per_group; ic++) {
                            for (ir = 0; ir < fy; ir++) {
                                cur_h = sy * ioh - py + dy * ir;
                                if (cur_h < 0 || cur_h >= h)
                                    continue;
                                for (is = 0; is < fx; is++) {
                                    cur_w = sx * iow - px + dx * is;
                                    if (cur_w < 0 || cur_w >= w)
                                        continue;
                                    i_idx = in * c * h * w + ig * c_per_group * h * w + ic * h * w +
                                            cur_h * w + cur_w;
                                    f_idx = ig * k_per_group * c_per_group * fy * fx + ik * c_per_group * fy * fx + ic * fy * fx +
                                            ir * fx + is;
                                    value += src[i_idx] * filter[f_idx];
                                }
                            }
                        }
                        dst[o_idx] = value;
                    }
                }
            }
        }
    }
}
```

```
// [bs, ic, ih, iw] & pack_size=8 => [bs, ic/8, ih, iw, 8]
// [bs, ic, ih, iw] & pack_size=4 => [bs, ic/4, ih, iw, 4]

// filter [oc, ic, kh, kw] & pack_in=8, pack_out=8 => [oc/8, ic/8, kh, kw, 8, 8]
// filter [oc, ic, kh, kw] & pack_in=4, pack_out=4 => [ic/4, ic/4, kh, kw, 4, 4]

// [bs, ]
```
# conv3d

# conv_depthwise
```
// [bs, ic, ih, iw] & pack_size=8 => [bs, ic/8, ih, iw, 8]
// [bs, ic, ih, iw] & pack_size=4 => [bs, ic/4, ih, iw, 4]

// filter [oc, ic/groups=1, kh, kw]
// filter [oc, 1, ih, iw] & pack_size=8 => [oc/8, ih, iw, 8]
// filter [oc, 1, ih, iw] & pack_size=4 => [ic/4, ih, iw, 4]

// output [bs, oc, oh, ow]
// output_trans [bs, oc/8, oh, ow, 8]
// output_trans [bs, oc/4, oh, ow, 4]
// [bs, oc/8, oh, ow, 8] => [bs, oc, oh, ow]
```
