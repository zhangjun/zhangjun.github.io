---
title: Paddle Lite framework
date: 2021-09-19 14:13:58
tags:
---

# core
## context
```
class KernelContext {
 public:
  template <typename ContextT>
  ContextT& As() {
    if (!ctx_.valid()) {
      ctx_.set<ContextT>();
    }
    return *ctx_.get_mutable<ContextT>();
  }

 private:
  Any ctx_;
};
```
```
class ContextScheduler {
 public:
  static ContextScheduler& Global() {
    static auto* x = new ContextScheduler;
    return *x;
  }
  std::unique_ptr<KernelContext> NewContext(
      TargetType target,
      /*only used for cuda context*/ int exec_stream_id = 0) {
    std::unique_ptr<KernelContext> ctx(new KernelContext);
    switch (target) {
      case TARGET(kHost):
        kernel_contexts_[TargetType::kHost].As<HostContext>().CopySharedTo(
            &ctx->As<HostContext>());
        break;
    }
    return ctx;
  } 
private:
  template <TargetType Type, typename ContextT>
  void InitContext() {
    kernel_contexts_[Type].As<ContextT>().InitOnce();
  }
  ContextScheduler() {
    InitContext<TargetType::kHost, HostContext>();
  }
private:
  std::map<TargetType, KernelContext> kernel_contexts_;
};
```
## op lite
```
class OpLite : public Registry {
 public:
  OpLite() = default;
  explicit OpLite(const std::string &type) : op_type_(type) {}
  explicit OpLite(const std::vector<Place> &valid_places)
      : valid_places_(valid_places) {}

  void SetValidPlaces(const std::vector<Place> &places) {
    VLOG(5) << "valid places " << valid_places_.size();
    valid_places_ = places;
  }
  virtual bool Run();
  // Indicate whether the Op runs only once or not
  virtual bool run_once() const { return false; }
  std::string Type() const { return op_type_; }

  // Link the external execution environ to internal context.
  bool Attach(const cpp::OpDesc &opdesc, lite::Scope *scope);

  template <typename T>
  inline void AttachParam(T *param) {
    op_param_ = static_cast<T *>(param);
  }
  // Create all the kernels for the valid targets.
  std::vector<std::unique_ptr<KernelBase>> CreateKernels(
      const std::vector<Place> &places, const std::string &kernel_type = "");

  Scope *scope() { return scope_; }

  // Assign op param to kernel.
  virtual void AttachKernel(KernelBase *kernel) = 0;
  void SetKernel(std::vector<std::unique_ptr<KernelBase>> &kernels) {  // NOLINT
    kernel_ = std::move(kernels.front());
    kernel_->SetContext(
        ContextScheduler::Global().NewContext(kernel_->target()));
  }

  KernelBase *GetKernel() {  // NOLINT
    return kernel_.get();
  }
  virtual ~OpLite() = default;
 protected:
  friend class mir::Node;
  friend class mir::SSAGraph;
 protected:
  Scope *scope_{nullptr};
  std::unique_ptr<KernelBase> kernel_;
  std::string op_type_;
  std::vector<Place> valid_places_;
  Place kernel_place_{TARGET(kHost), PRECISION(kFloat)};
  std::unique_ptr<OpInfo> op_info_;
  // todo: it's prefered to combine last_input_shapes and
  // last_input_lods into a single hash value to decrease
  // memory usage.
  std::vector<DDimLite> last_input_shapes{};
  std::vector<std::vector<std::vector<uint64_t>>> last_input_lods{};
  std::vector<DDimLite> last_output_shapes{};
  std::vector<std::vector<std::vector<uint64_t>>> last_output_lods{};
  mutable operators::ParamBase *op_param_{nullptr};

 private:
  // Infer Shape according to memory, if current input shapes are consistent
  // with that of previous inputs, output shapes of last time will be reused.
  bool InferShapeWithCache();
};
```
```
std::vector<std::unique_ptr<KernelBase>> OpLite::CreateKernels(
    const std::vector<Place> &places, const std::string &kernel_type) {
  std::vector<std::unique_ptr<KernelBase>> kernels;
  CHECK(!op_type_.empty()) << "op_type_ should be set first";

  auto pick_kernel = [&](const Place &place) {
    auto ks = KernelRegistry::Global().Create(
        op_type_, place.target, place.precision, place.layout);
    VLOG(5) << "pick kernel for " << op_info()->Type() << " "
            << place.DebugString() << " get " << ks.size() << " kernels";
    for (auto &&it : ks) {
      AttachKernel(it.get());
      kernels.emplace_back(std::move(it));
    }
  };

  if (!kernel_type.empty()) {
    Place place;
    std::string op_type, alias;
    KernelBase::ParseKernelType(kernel_type, &op_type, &alias, &place);
    pick_kernel(place);
    CHECK(!kernels.empty()) << "no kernel for kernel type " << kernel_type;
    return kernels;
  }

  std::set<Place> expanded_places(places.begin(), places.end());
  for (auto &place : places) {
    // Pick kernels those support any Precision and any DataLayout, For example:
    // kARM,kFloat,kNCHW -> kARM,kFloat,kAny; kARM,kAny,kNCHW; kARM,kAny,kAny
    expanded_places.insert(
        Place(place.target, place.precision, DATALAYOUT(kAny)));
    expanded_places.insert(Place(place.target, PRECISION(kAny), place.layout));
    expanded_places.insert(
        Place(place.target, PRECISION(kAny), DATALAYOUT(kAny)));
  }

  std::set<TargetType> targets;
  for (auto place : expanded_places) {
    pick_kernel(place);
    targets.insert(place.target);
  }

  VLOG(5) << "op " << op_type_ << " get " << kernels.size() << " kernels";
  return kernels;
}
```
```
/*
 * Operator Information, such as some description. It will be shared by all the
 * kernels of the same operator.
 */
class OpInfo : public cpp::OpDesc {
 public:
  OpInfo(const OpInfo &) = default;
  explicit OpInfo(const cpp::OpDesc &other) : cpp::OpDesc(other) {}
};
```
## op registry
```
class OpKernelInfoCollector {
 public:
  static OpKernelInfoCollector& Global() {
    static auto* x = new OpKernelInfoCollector;
    return *x;
  }
  void AddOp2path(const std::string& op_name, const std::string& op_path);
  void AddKernel2path(const std::string& kernel_name,
                      const std::string& kernel_path);
  
 private:
  std::map<std::string, std::string> op2path_;
  std::map<std::string, std::string> kernel2path_;
};
```
```
class OpLiteFactory {
 public:
  // Register a function to create an op
  void RegisterCreator(const std::string& op_type,
                       std::function<std::shared_ptr<OpLite>()> fun) {
    op_registry_[op_type] = fun;
  }

  static OpLiteFactory& Global() {
    static OpLiteFactory* x = new OpLiteFactory;
    return *x;
  }

  std::shared_ptr<OpLite> Create(const std::string& op_type) const {
    auto it = op_registry_.find(op_type);
    if (it == op_registry_.end()) return nullptr;
    return it->second();
  }

  std::string DebugString();

  std::vector<std::string> GetAllOps() const {
    std::vector<std::string> res;
    for (const auto& op : op_registry_) {
      res.push_back(op.first);
    }
    return res;
  }

 protected:
  std::map<std::string, std::function<std::shared_ptr<OpLite>()>> op_registry_;
};
```
```
class OpLiteRegistrar {
 public:
  OpLiteRegistrar(const std::string& op_type,
                  std::function<std::shared_ptr<OpLite>()> fun) {
    OpLiteFactory::Global().RegisterCreator(op_type, fun);
  }
  // Touch function is used to guarantee registrar was initialized.
  void touch() {}
};

class KernelFactory {
 public:
  // Register a function to create kernels
  void RegisterCreator(const std::string& op_type,
                       TargetType target,
                       PrecisionType precision,
                       DataLayoutType layout,
                       std::function<std::unique_ptr<KernelBase>()> fun) {
    op_registry_[op_type][std::make_tuple(target, precision, layout)].push_back(
        fun);
  }

  static KernelFactory& Global() {
    static KernelFactory* x = new KernelFactory;
    return *x;
  }

  /**
   * Create all kernels belongs to an op.
   */
  std::list<std::unique_ptr<KernelBase>> Create(const std::string& op_type) {
    std::list<std::unique_ptr<KernelBase>> res;
    if (op_registry_.find(op_type) == op_registry_.end()) return res;
    auto& kernel_registry = op_registry_[op_type];
    for (auto it = kernel_registry.begin(); it != kernel_registry.end(); ++it) {
      for (auto& fun : it->second) {
        res.emplace_back(fun());
      }
    }
    return res;
  }

  /**
   * Create a specific kernel. Return a list for API compatible.
   */
  std::list<std::unique_ptr<KernelBase>> Create(const std::string& op_type,
                                                TargetType target,
                                                PrecisionType precision,
                                                DataLayoutType layout) {
    std::list<std::unique_ptr<KernelBase>> res;
    if (op_registry_.find(op_type) == op_registry_.end()) return res;
    auto& kernel_registry = op_registry_[op_type];
    auto it = kernel_registry.find(std::make_tuple(target, precision, layout));
    if (it == kernel_registry.end()) return res;
    for (auto& fun : it->second) {
      res.emplace_back(fun());
    }
    return res;
  }

 protected:
  // Outer map: op -> a map of kernel.
  // Inner map: kernel -> creator function.
  // Each kernel was represented by a combination of <TargetType, PrecisionType,
  // DataLayoutType>
  std::map<std::string,
           std::map<std::tuple<TargetType, PrecisionType, DataLayoutType>,
                    std::list<std::function<std::unique_ptr<KernelBase>()>>>>
      op_registry_;
};

// Register Kernel by initializing a static KernelRegistrar instance
class KernelRegistrar {
 public:
  KernelRegistrar(const std::string& op_type,
                  TargetType target,
                  PrecisionType precision,
                  DataLayoutType layout,
                  std::function<std::unique_ptr<KernelBase>()> fun) {
    KernelFactory::Global().RegisterCreator(
        op_type, target, precision, layout, fun);
  }
  // Touch function is used to guarantee registrar was initialized.
  void touch() {}
};

class ParamTypeDummyRegistry {
 public:
  struct NewInstance {
    NewInstance() {}
    NewInstance& BindInput(const std::string& arg_name,
                           const ParamType& ptype) {
      return *this;
    }
    NewInstance& BindOutput(const std::string& arg_name,
                            const ParamType& ptype) {
      return *this;
    }
    NewInstance& SetVersion(const std::string& version) { return *this; }
    NewInstance& BindPaddleOpVersion(const std::string& op_type,
                                     int32_t version_id) {
      return *this;
    }
    bool Finalize() { return true; }
  };

 private:
  ParamTypeDummyRegistry() = default;
};
```

# 注册机制
## op注册
```
#define REGISTER_LITE_OP(op_type__, OpClass)                                   \
  static paddle::lite::OpLiteRegistrar op_type__##__registry(                  \
      #op_type__, []() {                                                       \
        return std::unique_ptr<paddle::lite::OpLite>(new OpClass(#op_type__)); \
      });                                                                      \
  int touch_op_##op_type__() {                                                 \
    op_type__##__registry.touch();                                             \
    OpKernelInfoCollector::Global().AddOp2path(#op_type__, __FILE__);          \
    return 0;                                                                  \
  }
```
## kernel 注册
```
// Register a kernel.
#define REGISTER_LITE_KERNEL(                                                 \
    op_type__, target__, precision__, layout__, KernelClass, alias__)         \
  static paddle::lite::KernelRegistrar                                        \
      op_type__##target__##precision__##layout__##alias__##_kernel_registry(  \
          #op_type__,                                                         \
          TARGET(target__),                                                   \
          PRECISION(precision__),                                             \
          DATALAYOUT(layout__),                                               \
          []() {                                                              \
            std::unique_ptr<KernelClass> x(new KernelClass);                  \
            x->set_op_type(#op_type__);                                       \
            x->set_alias(#alias__);                                           \
            return x;                                                         \
          });                                                                 \
  int touch_##op_type__##target__##precision__##layout__##alias__() {         \
    op_type__##target__##precision__##layout__##alias__##_kernel_registry     \
        .touch();                                                             \
    OpKernelInfoCollector::Global().AddKernel2path(                           \
        #op_type__ "," #target__ "," #precision__ "," #layout__ "," #alias__, \
        __FILE__);                                                            \
    return 0;                                                                 \
  }                                                                           \
  ParamTypeRegistry(                                                          \
      op_type__, target__, precision__, layout__, KernelClass, alias__)
```
# program
## scope
```
class Scope final {
 public:
  Scope()
      : kids_lock_{new lite::fluid::RWLock},
        vars_lock_{new lite::fluid::RWLock},
        rwlock_{new lite::fluid::RWLock} {}
  // delete below two functions to allow pybind to recognise it cannot make a
  // copy
  // link:
  // https://stackoverflow.com/questions/53807248/pybind11-returning-a-pointer-to-a-container-of-unique-ptr
  Scope(const Scope&) = delete;
  Scope& operator=(const Scope&) = delete;
  ~Scope();

  Scope& NewScope() const;

  Variable* Var(const std::string& name);

  Variable* LocalVar(const std::string& name);

  Variable* FindVar(const std::string& name) const;

  Variable* FindLocalVar(const std::string& name) const;

  const Scope* parent() const { return parent_; }

  // Get attribute params stored in parent scopes.
  std::vector<std::string> AttributeVarNames() const;
  // Following the legacy scope interface.
  std::vector<std::string> LocalVarNames() const;

  /// ------------------------------------- helper functions for Tensor
  /// ----------------------------------
  // Create a Tensor variable. This will create a new Variable called `name`.
  Tensor* NewTensor(const std::string& name) {
    auto* var = Var(name);
    return var->GetMutable<Tensor>();
  }

  const Tensor* FindTensor(const std::string& name) {
    auto* var = FindVar(name);
    if (!var) return nullptr;
    return &var->Get<Tensor>();
  }

  Tensor* FindMutableTensor(const std::string& name) {
    auto* var = FindVar(name);
    if (!var) return nullptr;
    return var->GetMutable<Tensor>();
  }

  std::vector<Tensor>* NewTensorList(const std::string& name) {
    auto* var = Var(name);
    return var->GetMutable<std::vector<Tensor>>();
  }

  const std::vector<Tensor>* FindTensorList(const std::string& name) {
    auto* var = FindVar(name);
    if (!var) return nullptr;
    return &var->Get<std::vector<Tensor>>();
  }

  std::vector<Tensor>* FindMutableTensorList(const std::string& name) {
    auto* var = FindVar(name);
    if (!var) return nullptr;
    return var->GetMutable<std::vector<Tensor>>();
  }

 private:
  // Scope in `kids_` are owned by this class.
  mutable std::list<Scope*> kids_;
  const Scope* parent_{nullptr};
  std::map<std::string, std::unique_ptr<Variable>> vars_;
  std::unique_ptr<lite::fluid::RWLock> kids_lock_{nullptr};
  std::unique_ptr<lite::fluid::RWLock> vars_lock_{nullptr};
  std::unique_ptr<lite::fluid::RWLock> rwlock_{nullptr};
};
```
# optimize
## optimizer
```
/*
 * lite::Optimizer optimize a program. It utilize the mir passes to analysis the
 * program and export an optimized program.
 * Example :
 *       // (1) Create an optimizer
 *       Optimizer optim(valid_places, kernel_pick_factor);
 *       // (2) add an optimizer method
 *       optim.AddPass("post_quant_dynamic_pass");
 *       // (3) analysis a program to export an optimized program
 *       auto program_ = optim.Run(std::move(program));
 */
class Optimizer {
 public:
  Optimizer(const std::vector<Place>& valid_places,
            core::KernelPickFactor kernel_pick_factor)
      : valid_places_(valid_places), kernel_pick_factor_(kernel_pick_factor) {
    CHECK(!valid_places.empty()) << "At least one valid_place should be set";
  }

  // Append a pass to the optimizer.
  void AddPass(const std::string& pass_name);
  // Optimize a program to generate a runtime program.
  std::unique_ptr<RuntimeProgram> Run(Program&& program);

 protected:
  // Run all the added passes.
  void ApplyPasses(std::vector<std::unique_ptr<mir::SSAGraph>>* graphes);

  // Generate the optimized runtime program.
  std::unique_ptr<RuntimeProgram> GenRuntimeProgram(
      std::vector<std::unique_ptr<mir::SSAGraph>>* graphs);

  void InitTargetTypeTransformPass();
  void InitControlFlowOpUnusedInputsAndOutputsEliminatePass();
  void InitControlFlowOpSharedInputsAndOutputsPlaceSyncPass();
  void SpecifyKernelPickTactic(core::KernelPickFactor factor);
  Scope* exec_scope() { return exec_scope_; }

 private:
  std::vector<Place> valid_places_;
  Scope* exec_scope_{};
  std::vector<mir::Pass*> passes_;
  std::vector<std::unique_ptr<mir::SSAGraph>> graphs_;
  core::KernelPickFactor kernel_pick_factor_;
};
```