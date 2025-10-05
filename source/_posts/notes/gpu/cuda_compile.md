---
title: cuda compile
tags: [CUDA, Compilation, GPU]
excerpt: CUDA编译技术详解，包括nvcc编译器参数配置、虚拟架构和真实架构的区别、PTX和CUBIN文件生成，以及多架构兼容性编译策略。
---

## nvcc compile
https://blog.csdn.net/weixin_36670529/article/details/105910109
- -arch=compute_80, 虚拟GPU架构编译成ptx
- -code=sm_80, 真实GPU架构编译成cubin
![image](https://user-images.githubusercontent.com/1312389/188297745-ed683d41-8cf4-4d1c-a103-03683bcf44df.png)

![image](https://user-images.githubusercontent.com/1312389/188297793-283e6aad-3946-4e60-96fd-f0b67036e55f.png)
`nvcc xxx.cu -arch=compute_60 -code=sm_60` 即达成上图效果。对应真实架构sm_60的二进制的指令被嵌入到最用的可执行程序或者库文件，由于没有嵌入ptx，无法进行即时编译运行在sm > 60 GPU 上。

![image](https://user-images.githubusercontent.com/1312389/188298061-ae273d4e-894e-4dac-b11a-12254a2a19de.png)
