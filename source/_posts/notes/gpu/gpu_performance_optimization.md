---
title: GPU Performance Optimization
tags: [CUDA, GPU, Performance, Register Spilling, Active Warp]
excerpt: GPU performance optimization concepts including register spilling (causes, effects, overflow path) and active warp (occupancy, latency hiding, warp scheduler).
---

## Register Spilling

- **Definition**
    - When the live variables in a kernel exceed the capacity of the SM’s **register file**, the compiler must spill excess values to **off‑chip memory**. Spilled data goes through L1 → L2 → **local memory** (a region of global memory per thread).
- **Causes**
    - **High register usage per thread**: complex expressions, many locals, large structs, or unrolling that keeps many values live.
    - **Large blocks or high occupancy** with many threads per block, so total registers (threads × registers/thread) exceed the SM’s limit.
- **Effects**
    - **Slow access**: local memory has global-memory latency and bandwidth; each spilled load/store is much slower than a register access.
    - **Extra traffic**: more transactions on the memory bus, which can hurt bandwidth and L1/L2 utilization.
    - **Worse occupancy**: high per-thread register use can reduce the number of concurrent blocks/warps, which reduces latency hiding.
- **Overflow path**
    - Register file (on-chip) → L1 cache (overflow) → L2 cache → **local memory** (main memory, per-thread).
- **How to reduce / avoid**
    - **Use fewer registers per thread**: simplify expressions, avoid unnecessary locals, reduce loop unrolling when it increases register pressure.
    - **Restrict `-maxrregcount`** (or equivalent) to cap register use and force spilling only when needed; sometimes a little spilling with higher occupancy is better than low occupancy.
    - **Reduce thread block size** or **reduce occupancy** to stay within the SM’s total register budget.
    - **Inspect with `nvcc --ptxas-options=-v`** or `ncu`/`nsys` to see `spills` and `local memory` usage.

## Active Warp

- **Definition**
    - An **active warp** is a warp (32 threads) that has been launched on an SM and has at least one instruction left to execute. The number of active warps per SM is limited by registers, shared memory, and max warps per SM.
- **Role in performance**
    - **Latency hiding**: when one warp stalls (e.g., waiting for global memory), the **warp scheduler** can run other active warps. More active warps → better overlap of memory latency with compute.
    - **Occupancy** = (active warps per SM) / (max warps per SM). Higher occupancy usually means more opportunity to hide latency, but it is not always better if it forces register spilling or worsens cache behavior.
- **Limits**
    - **Register file**: `(registers per thread) × (threads per block) × (blocks per SM)` cannot exceed the SM’s total registers. More registers per thread → fewer blocks/warps per SM.
    - **Shared memory**: total shared memory per block × blocks per SM ≤ shared memory per SM. Large `__shared__` usage can lower the maximum blocks per SM.
    - **Max warps per SM** and **max blocks per SM** (device-specific). The **actual** active warps are the minimum allowed by registers, shared memory, and these caps.
- **Warp scheduler**
    - Each SM has one or more warp schedulers. Each cycle, a scheduler picks a warp that is **ready** (not stalled) and issues one or two instructions (depending on the architecture). Warps waiting on memory or other long-latency operations do not get scheduled until the operation completes.
- **Practical tips**
    - **Tune block size** (and thus warps per block) so that occupancy is not unnecessarily low; often 2–4 warps per block or 50–75% occupancy is a reasonable target before register/shared pressure bites.
    - **Check with `ncu` or `nsys`**: occupancy, active warps per SM, warp execution efficiency, and stall reasons (e.g., memory throttle, math throttle).
    - **Balance with register/shared use**: sometimes lower occupancy with fewer spills or better cache reuse is faster than maximum occupancy with heavy spilling.
