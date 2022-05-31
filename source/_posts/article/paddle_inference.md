<!--
 * @Author: Zhang Jun ewalker@live.cn
 * @Date: 2021-09-05 18:00:17
 * @LastEditors: Zhang Jun ewalker@live.cn
 * @LastEditTime: 2022-06-01 00:26:44
 * @FilePath: /undefined/Users/apple/Downloads/zhangjun/github/zhangjun.github.io/source/_posts/article/paddle_inference.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
---
title: paddle inference
date: 2021-03-16 03:32:13
tags:
---
paddle inference学习记录

# 代码解析
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

## 待添加
framework::ProgramDesc
framework::Executor* executor
framework::Scope* scope

# paddle inference 执行逻辑
```
paddle/fluid/framework/naive_executor.cc#L41
```
```
void NaiveExecutor::Run() {
#ifdef PADDLE_WITH_MKLDNN
  platform::AttachPointerHashToMKLDNNKey(this, place_);
  platform::RegisterModelLayout(ops_, place_);
#endif
  platform::ScopedFlushDenormal flush;
  for (auto &op : ops_) {
    VLOG(4) << std::this_thread::get_id() << " run "
            << op->DebugStringEx(scope_) << " on scope " << scope_;
    op->SetIsCalledByExecutor(false);
    op->Run(*scope_, place_);
  }
}
```

```
paddle/fluid/framework/operator.cc#L204
```
```
void OperatorBase::Run(const Scope& scope, const platform::Place& place) {
}
```