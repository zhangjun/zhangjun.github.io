---
title: gpu new features
excerpt: NVIDIA GPU新特性介绍，包括V100的Volta SIMT模型、Cooperative Groups，以及A100的异步拷贝、异步屏障、任务图加速和2:4结构化稀疏等先进技术。
---

## V100 新特性
- Volta SIMT Model
<img width="700" alt="image" src="https://user-images.githubusercontent.com/1312389/184500671-18368901-0704-4489-b35b-58251427d9c9.png">
__syncwarp() to force reconvergence
- Cooperative Groups

## A100 新特性
- Asynchronous copy
<img width="1073" alt="image" src="https://user-images.githubusercontent.com/1312389/184522235-42f463ff-5343-4b41-8a69-6b62e49afcb5.png">

- Asynchronous barrier
- Task graph acceleration
- 2:4 structured sparsity
![image](https://user-images.githubusercontent.com/1312389/184498718-d310b435-e662-4a30-9433-0932d9e701dc.png)