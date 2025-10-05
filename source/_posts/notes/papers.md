---
title: paper lists
excerpt: 深度学习相关论文列表，涵盖推理优化、分布式训练、通信压缩、量化技术等领域的重要论文，包括PipeDream、梯度压缩、量化算法等核心技术。
---

## inference

## train
### PS
- [Accelerating Collective Communication in Data Parallel Training across Deep Learning Frameworks](https://www.usenix.org/system/files/nsd()i22spring_prepub_romero.pdf)
### parallel training
- [PipeDream: Generalized Pipeline Parallelism for DNN Training](https://www.pdl.cmu.edu/PDL-FTP/BigLearning/sosp19-final271.pdf)
- https://insujang.github.io/2022-06-11/parallelism-in-distributed-deep-learning/
### communication
- [Compressed Communication for Distributed Deep Learning: Survey and Quantitative Evaluation](https://repository.kaust.edu.sa/bitstream/handle/10754/662495/gradient-compression-survey.pdf?sequence=1&isAllowed=y)
- [Efficient Sparse Collective Communication and its application to Accelerate Distributed Deep Learning](https://conferences.sigcomm.org/sigcomm/2021/files/papers/3452296.3472904.pdf)

## quantization
- [Quantization and Training of Neural Networks for Efficient Integer-Arithmetic-Only Inference](https://arxiv.org/pdf/1712.05877.pdf)

   https://blog.csdn.net/qq_19784349/article/details/82883271
   $r=S(q-Z)$  => $q=round(\frac{r}{S}+Z)$;   S - scale, Z - zero-point
   $\Large{S=\frac{val_{max}-val_{min}}{2_{bit\_length}-1}}$
   $\Large{Z=round(-\frac{val_{min}}{S})}$   
   <img width="342" alt="image" src="https://github.com/zhangjun/zhangjun.github.io/assets/1312389/e6cd519b-8228-4540-ab24-73e6a796a3e6">

## llm