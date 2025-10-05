---
title: torch compile
excerpt: PyTorch 2.0 torch.compile编译优化技术详解，包括TorchDynamo、AOT Autograd、PrimTorch和TorchInductor组件，以及torch.compile API的使用方法和代码解析。
---

## torch.compile
TorchDynamo, AOT Autograd, PrimTorch, and TorchInductor
![image](https://github.com/zhangjun/resume/assets/1312389/dedbad08-438b-44c6-8826-52a19c04291b)
https://pytorch.org/get-started/pytorch-2.0/#user-experience
![image](https://github.com/zhangjun/resume/assets/1312389/b493fed6-567d-4162-8ca2-7727b49eaad3)

### TorchDynamo
torch.compile -> TorchDynamo -> FX graphs
torch dynamo通过jit方式将任意python代码编译为FX graphs，允许不同后端进一步进行优化。
torch._dynamo.optimize()
![image](https://github.com/zhangjun/resume/assets/1312389/4aa6e068-474e-4850-9c3d-1390a9135c1f)

## torch compile 代码解析
```python
def compile(model: Optional[Callable] = None, *,
            fullgraph: builtins.bool = False,
            dynamic: Optional[builtins.bool] = None,
            backend: Union[str, Callable] = "inductor",
            mode: Union[str, None] = None,
            options: Optional[Dict[str, Union[str, builtins.int, builtins.bool]]] = None,
            disable: builtins.bool = False) -> Callable:
    mode = "default"
    if backend == "inductor":
        backend = _TorchCompileInductorWrapper(mode, options, dynamic)
    else:
        backend = _TorchCompileWrapper(backend, mode, options, dynamic)

    return torch._dynamo.optimize(backend=backend, nopython=fullgraph, dynamic=dynamic, disable=disable)(model)

```