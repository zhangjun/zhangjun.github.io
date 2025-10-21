---
title: gpu instruction throughput
tags: [GPU, Performance, Instruction]
excerpt: GPU instruction throughput and latency analysis, detailing performance characteristics of different instruction types and instruction execution capabilities per SM, providing important reference data for GPU programming optimization.
---

# Instruction Throughput
<img width="1119" alt="image" src="https://user-images.githubusercontent.com/1312389/184467524-d5054598-c80d-4fec-bb21-8a61ed6fcd64.png">

# Instruction Latencies and Instructions/SM
<img width="731" alt="image" src="https://user-images.githubusercontent.com/1312389/184467471-803dffdd-8be4-4402-9283-f829134bdb98.png">


![alt text](image.png)

## Little's law
所需线程数量 = 延迟*吞吐量
### Arithmetic Instruction Latency
### Memory Instruction Latency
每个时钟周期的读取字节数 = 内存带宽 / 时钟频率