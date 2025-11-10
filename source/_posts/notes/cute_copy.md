---
title: Cute Copy
date: 2025-11-11 00:48:29
tags: [cute]
---

## 异步拷贝概念
### 
- 从全局内存加载数据到共享内存 [global mem] -> [L2 cache] -> [L1 cache] -> [register] -> [shared mem]
- cp.copy.cg [global mem] -> [L2 cache] -> [shared mem]
- cp.copy.ca [global mem] -> [L2 cache] -> [L1 cache] -> [shared mem]

## 不同架构下内存拷贝
### Ampere(SM80)
- cp.async
```cpp
cp.async.ca.shared{::cta}.global{.level::cache_hint}{.level::prefetch_size}
						[dst], [src], cp-size{, src-size}{, cache-policy} ;
cp.async.cg.shared{::cta}.global{.level::cache_hint}{.level::prefetch_size}
						[dst], [src], 16{, src-size}{, cache-policy} ;
cp.async.ca.shared{::cta}.global{.level::cache_hint}{.level::prefetch_size}
						[dst], [src], cp-size{, ignore-src}{, cache-policy} ;
cp.async.cg.shared{::cta}.global{.level::cache_hint}{.level::prefetch_size}
						[dst], [src], 16{, ignore-src}{, cache-policy} ;
						
.level::cache_hint = { .L2::cache_hint }
.level::prefetch_size = { .L2::64B, .L2::128B, .L2::256B }
cp-size = { 4, 8, 16 }

TS const* gmem_ptr    = &gmem_src;
uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_dst);
asm volatile("cp.async.ca.shared.global.L2::128B [%0], [%1], %2;\n"
    :: "r"(smem_int_ptr),
        "l"(gmem_ptr),
        "n"(sizeof(TS)));
```
- 同步机制
同步机制有两种：Async Group; mbarrier
```cpp
// Establishes an ordering w.r.t previously issued cp.async instructions. Does not block.
cp_async_fence()
{
#if defined(CUTE_ARCH_CP_ASYNC_SM80_ENABLED)
  asm volatile("cp.async.commit_group;\n" ::);
#endif
}

// /// Blocks until all but N previous cp.async.commit_group operations have committed.
template <int N>
CUTE_HOST_DEVICE
void
cp_async_wait()
{
  if constexpr (N == 0) {
    asm volatile("cp.async.wait_all;\n" ::);
  } else {
    asm volatile("cp.async.wait_group %0;\n" :: "n"(N));
  }
}
```
cp.async.wait_all = cp.async.commit_group + cp.async.wait_group 0

async group是per thread的。一个async group内的异步操作完成顺序是无序的，async group之间完成顺序取决于提交顺序。

### 


### https://www.zhihu.com/column/c_1669290006261825536