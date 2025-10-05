---
title: cuda memory
tags: [CUDA, GPU, Memory]
excerpt: CUDA memory hierarchy explained, including register file, L1 cache, shared memory, constant cache, L2 cache, global memory, local memory, texture and constant memory characteristics and usage.
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