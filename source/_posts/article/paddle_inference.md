---
title: paddle inference
date: 2021-03-16 03:32:13
tags:
---
paddle inference学习记录

#代码解析
paddle inference代码位于paddle/fluid/inference下面。

## engine基类
```
class EngineBase {
 public:
  using DescType = ::paddle::framework::proto::BlockDesc;
  // Build the model and do some preparation, for example, in TensorRT, run
  // createInferBuilder, buildCudaEngine.
  virtual void Build(const DescType& paddle_model) = 0;
  // Execute the engine, that will run the inference network.
  virtual void Execute(int batch_size) = 0;
  virtual ~EngineBase() {}
}; 
```
