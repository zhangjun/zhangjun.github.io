---
title: cuda memory
tags: [CUDA, GPU, Memory]
excerpt: CUDA memory hierarchy explained, including register file, L1 cache, shared memory, constant cache, L2 cache, global memory, local memory, texture and constant memory characteristics and usage. Memory access patterns for global and shared memory, and optimization techniques (SoA vs AoS, vectorized loads, __ldg, broadcast, padding, bank-aware layout, occupancy, TMA, async copy).
---

## Memory 
- Register File
- L1 cache
    - on-chip storage，serves as the overflow region when the amount of active data exceeeds what an SM's register file can hold
- Shared Memory
    - physically resides in the same memory as the L1 cache，accessed by any thread in a thread block
- Constant Caches
    - store variables declared as read-only constants in global memory，can be read by any thread in a thread block. Used to broadcast a single constant value to all the threads in a warp.
- L2 Cache 
    - on-chip cache for retaining copies of the data that travel back and forth between the SMs and main memory. shared by all the SMs. The L2 cache is also situated in the path of data moving on or off the device via PCIe or NVLink. 
- Global Memory
- Local Memory
    - corresponds to specially mapped regions of main memory that are assigned to each SM. Whenever "register spilling" overflows the L1 cache on a particular SM, the excess data are further offloaded to L2, then to "local memory".
- Texture and Constant Memory
    - regions of main memory that are treated as read-only by the device.  accessed by any thread in a thread block. Texture memory is cached in L1, while constant memory is cached in the constant caches.

## Memory Access Patterns
- Aligned Memory access
- Coalesced Memory access

### Global Memory

- **Coalesced access**
    - threads in a warp (32 threads) access consecutive memory addresses. Ideally: 32 threads × 4 bytes = 128 bytes in one transaction; best when the warp’s first thread address is 128-byte aligned.
- **Uncoalesced access**
    - scattered or strided access causes multiple transactions per warp and wastes bandwidth. Stride-N access (e.g. `a[threadIdx.x + blockIdx.x * blockDim.x * N]`) fetches N× more data than needed.
- **Transaction size**
    - hardware issues 32-, 64-, or 128-byte transactions. Misalignment or non-sequential access can turn one logical access into several transactions.
- **Rule of thumb**
    - have thread `i` read/write `base + i` (or similar contiguous layout) so the warp’s accesses form one or few contiguous segments.

### Shared Memory

- **Banks**
    - shared memory is split into 32 banks (e.g. 4-byte banks). Successive 4-byte words map to successive banks; bank 0–31 then repeats.
- **Bank conflict**
    - when two or more threads in a warp access different addresses in the same bank, accesses are serialized. Same-address (broadcast) access is supported and does not conflict.
- **Conflict-free patterns**
    - no conflict when each bank is accessed by at most one thread in the warp (e.g. `shared[threadIdx.x]` with `threadIdx.x` in 0–31).
- **Strided access**
    - e.g. `shared[threadIdx.x * 2]` makes threads 0 and 16 use the same bank → 2-way conflict; higher strides can cause 32-way conflicts.
- **Padding**
    - adding padding (e.g. one extra element per row in 2D `shared`) can break problematic strides and avoid bank conflicts when the access pattern is fixed.

### Optimization

- **Data layout: SoA vs AoS**
    - **Structure of Arrays (SoA)**: store each field in a separate contiguous array (`x[i]`, `y[i]`, `z[i]`). Warps access consecutive addresses → coalesced, fewer transactions.
    - **Array of Structures (AoS)**: `struct { x,y,z } arr[i]` causes strided access when only one field is needed → prefer SoA when each thread uses one field, or ensure field offsets align to coalescing.

- **Vectorized loads (float4, int4, etc.)**
    - Use `float4`, `int4` (or `uint4`) for 16-byte aligned, contiguous data. One instruction can issue a 128-byte transaction for a full warp, improving bandwidth. Requires 16-byte alignment of the base pointer.

- **Read-only global: `__ldg()` and const memory**
    - `__ldg(ptr)` reads through the texture cache; use for read-only, irregular, or broadcast-like access. Constant memory (`__constant__`) suits small read-only data and same-index reads across a warp (broadcast, no bank conflict).

- **Broadcast**
    - When all threads in a warp (or block) read the **same address**: **shared memory** same-address access is a single read broadcast to the warp (no bank conflict); **constant memory** and **texture** are also optimized for broadcast. Place scalar or common read-only data in `__constant__` or a single shared slot instead of each thread reading from global. Avoid replicating the same load across threads when one load + broadcast is possible.

- **Minimize strided and random access**
    - Replace stride-N patterns with transpose or blocked layout so each warp touches consecutive elements. If gather/scatter is unavoidable, batch and align where possible; consider `__ldg()` for reads.

- **Padding**
    - **Shared memory**: pad the leading dimension in 2D layouts (e.g. `dim + 1`) so stride-`blockDim.x` access maps to different banks and avoids N-way bank conflicts. Example: `shared[threadIdx.x + threadIdx.y * (blockDim.x + 1)]`. For 1D, pad to a multiple of 32 or 64 when it simplifies conflict-free indexing.
    - **Global / alignment**: pad structs or rows to 128-byte (or 16-byte) boundaries when it improves coalescing or reduces transactions for vectorized loads.

- **Shared memory: bank-aware layout**
    - Prefer `shared[threadIdx.x]` (linear) or `shared[threadIdx.x + threadIdx.y * (blockDim.x + 1)]` with padding so each bank is used by at most one thread in the warp; avoid stride-`blockDim.x` without padding.

- **Occupancy and spilling**
    - High register and shared usage lowers occupancy and can cause **register spilling** into local memory (slow, uncoalesced). Use `--ptxas-options=-v` or `nvcc -Xptxas -v` to check; reduce per-thread state or shared size to improve occupancy and hide latency.

- **New architecture: TMA and async copy**
    - **TMA (Tensor Memory Accelerator, Hopper SM90+)**: hardware unit for asynchronous bulk copy of 2D/3D regions between global and shared memory. A single TMA descriptor can move large tiles (e.g. 16×16, 32×32, 64×64) without per-thread load/store instructions, improving bandwidth and freeing registers. Use `cp.async.bulk.tensor` (PTX) or `cuda::memcpy_async` (C++) with TMA for tiled matmul, convolutions, and block-level data movement.
    - **Async copy (`cp.async`, Ampere SM80+)**: `cp.async` / `cuda::memcpy_async` copies from global to shared **without blocking** the thread. Use `cp.async.commit_group` and `cp.async.wait_group` (or `cuda::memcpy_async` with `cuda::barrier`) to pipeline multiple copies with compute (double buffering, software pipelining). Overlaps memory transfer and arithmetic, hides latency, and reduces repeated global reads. Requires `-arch=sm_80` or higher.