---
title: 大模型训练优化
tags: [Large Model, Training, Optimization]
excerpt: 大模型训练优化技术全面解析，包括Megatron框架、计算优化（OP融合、混合精度、通信融合）、显存优化（重计算、Offload）、并行优化（数据并行、模型并行、流水线并行）等核心技术。
---

## Megatron
![image](https://github.com/zhangjun/resume/assets/1312389/10f7f403-031c-42e3-af47-1360065f831a)

## 计算优化
### OP 融合
组合不同的算子以减少运算和空间开销。
将模型网络中顺序执行的多个OPs进行融合能够减少OP 调度的开销，提升训练速度。目前Fleet 中支持如下3种的OP 融合：
* fuse_all_optimizer_ops：表明是否融合(fuse) 是否融合 optimizer_op，仅对部分 optimizer 可用（SGD、Adam和Momentum）。
* fuse_elewise_add_act_ops：表明是否融合(fuse) elementwise_add_op和activation_op。
* fuse_bn_act_ops：表明是否融合(fuse) batch_norm_op 和 activation_op。
### 自动混合精度
* Dynamic loss scaling：在AMP训练过程中，为了避免精度下溢，每训练一定数量批次的数据，就将Loss放大指定倍数。如果Loss在放大过程中发生上溢，则可以再缩小一定倍数，确保整个训练过程中，梯度可以正常收敛。
* op黑白名单
### 通信融合
每次触发通信都会有一些额外的操作（如先建立连接等），减少这些额外的操作将对性能有很大帮助。如果能够将多次通信的内容先拼接成连续的数据，然后在一次通信内全部发送/接收，那么将会更充分的利用硬件资源，获得更大的性能提升。AllReduce 融合默认情况下会将同一layer中参数的梯度的多个AllReduce操作合并成一个。 比如对于 fc 中有Weight和Bias两个参数，打开该选项之前，需要两次AllReduce操作；打开该选项之后，只用一次AllReduce 操作。这样可以减少梯度同步时的通信耗时。
### 通信计算重叠（Overlap）
计算和IO放入不同流中，实现执行时间上重叠
### 通信拓扑优化
分层通信思想，ring-allreduce,Double binary trees等多种拓扑结构
### 通信频率优化
用于低带宽场景（如： 公有云上训练，联邦训练）中，梯度同步在低带宽网络下的延迟成为训练速度的主要瓶颈
* DGC Deep Gradient Compression
使用深度梯度压缩选择重要梯度进行通信来减少通信量，降低对通信带宽的依赖。
DGC 论文中的 预热训练 (warming up training), 动量修正 (Momentum Correction), 局部梯度修剪 (local gradient clipping), 动量因子掩藏 (Momentum factor masking) 等策略， 和 正则化项修正 (Weight Decay Correction) 避免稀疏梯度通信训练带来的最终模型精度损失。
* LocalSGD

## 显存优化
###  Recompute
#### Forward Recomputation Backpropagation
FRB：该策略通过清除正向计算过程中的中间计算结果，来降低训练过程中使用的存储空间，从而确保硬件有足够的内存做更大batch Size 的训练。
#### Recompute-Offload
Recompute-Offload 原理大致可以分为两步：
* Forward： 当checkpoint在前向中被生成后，将其卸载(Offload)到Host 内存中，让其所占据的显存可以被释放。
* Backward：当checkpoint在反向中被重新调用之前，将其预取(Pre-fetch) 回显存中，完成之后的重计算。

## 并行优化
https://colossalai.org/docs/concepts/paradigms_of_parallelism/
https://www.zhihu.com/question/508671222/answer/2290801813
https://www.cnblogs.com/marsggbo/p/16871789.html
### Data Parallel
all-reduce
<img src="https://github.com/zhangjun/zhangjun.github.io/assets/1312389/09dead4f-0a40-4347-a3c5-978fdc0d4051" width="250"/>
### Model Parallel
#### Tensor Parallel
all-gather
#### Pipeline Parallel
model is split by layer into several chunks, each chunk is given to a device.
![image](https://github.com/zhangjun/zhangjun.github.io/assets/1312389/dc9f79c9-bd39-4b2f-8c01-30cd3d0c844d)
<img src="https://github.com/zhangjun/zhangjun.github.io/assets/1312389/b52e24e0-9f6d-4800-8ae5-8852672908ec" width="250"/>
### Optimizer-Level Parallel
### 重计算(Recomputation or Checkpointing)
### 零冗余优化器 (Zero REdundancy Optimizer)
### 专家并行 MOE
<img src="https://github.com/zhangjun/zhangjun.github.io/assets/1312389/dda0a6f1-5594-4f66-93be-86e1374e893c" width="400"/>


## 高级优化
### 4D混合并行
|并行方式|优点|缺点|
|--|--|--|
|数据并行|并行加速比最高|显存占用高|
|模型并行|单个算子拆分到多个硬件，适合机器内相同多个硬件加速|通信量高|
|流水线并行|适合超大规模训练|设备利用率、计算效率差，可以用micro-batch优化|
|Sharding|||

总训练速度 和 （单卡速度*卡数*多卡加速比）有关。 单卡速度由数据读取和计算速度决定，多卡加速比由计算/通信效率决定。

对于混合并行：假设每个mini-batch切分为micro_step个micro batches，每个micro-batch的batch size为micro_bsz，并假设流水线的级数为pp_num。
* 对于sharding，每个micro step需要对参数进行2次broadcast和1次reduce操作，因此每个micro step中总的通信量为2*M+M=3M,其中M为参数量大小。
* 对于数据并行，每个micro step需要使用allreduce sum操作同步当前机器上所有参数的梯度信息。
* 假设模型参数在所有流水线间均匀切分，那么每个流水线级中包含的参数量为M/pp_num，因此每个micro step总的通信量为2M/pp_num，其中2表示allreduce sum的通信因子。对于流水线并行，每个micro step传输的通信量为流水线相邻层间activation的大小；对于Transformer类模型，相邻层间的activation大小为hid_size * micro_bsz * seq_len；其中，hid_size表示模型参数隐层大小，seq_len表示模型参数序列长度，micro_bsz表示每个micro的batch size。
* 对于模型并行，每个Transformer Encoder层包含两次allreduce sum通信，每次通信量大小为hid_size*micro_bsz*seq_len;由于结合流水线并行，每个流水线级中包含的Transformer Encoder的层数为L/pp_num，其中，其中L表示模型总的Transformer Encoder的层数，其余各参数的意义同上；因此，模型并行配置下每台机器内部每个micro step的通信总量为4L*(hid_size*micro_bsz*seq_len)/pp_num，其中因子4表示allreduce_sum通信因子2与每个Transformer Encoder层包含两次allreduce sum通信次数的乘积。
