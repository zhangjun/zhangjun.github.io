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
  auto dev_id = place.device;
  platform::SetDeviceId(dev_id);
  auto op_name = platform::OpName(outputs_, Type());
  RunImpl(scope, place);
}
```
```
void OperatorWithKernel::RunImpl(const Scope& scope,
                                 const platform::Place& place,
                                 RuntimeContext* runtime_ctx) const {
  platform::DeviceContextPool& pool = platform::DeviceContextPool::Instance();
  auto* dev_ctx = pool.Get(place);
  auto exe_ctx = ExecutionContext(*this, scope, *dev_ctx, *runtime_ctx);
  // using cache
  if (kernel_type_.get()) {
    dev_ctx = pool.Get(kernel_type_->place_);
  }
  {
    impl_ =
        new CacheImpl(new phi::KernelContext(),
                          new RuntimeInferShapeContext(*this, *runtime_ctx));
    BuildPhiKernelContext(*runtime_ctx, dev_ctx, impl_->getKernelContext());

    (*pt_kernel_)(impl_->getKernelContext());
  }
}
```