---
title: TensorRT
date: 2022-01-06 07:47:06
tags: GPU, TensorRT
---
[TOC]
# introduction
![TensorRT](https://user-images.githubusercontent.com/1312389/150677468-932f4721-78ee-4789-936d-2e0dd4c13f4c.png)

# install 
可以使用三种方式进行安装，包括
* container 形式进行安装，下载[NGC container](http://ngc.nvidia.com/); 
* debian 形式安装
* pip 形式进行安装

## container 形式安装
下载https://github.com/NVIDIA/TensorRT/blob/main/docker/ubuntu-18.04.Dockerfile
```docker build -f ubuntu-18.04.Dockerfile --build-arg CUDA_VERSION=11.4.3 --tag=tensorrt-ubuntu .```

## debian 形式安装
## pip形式进行安装
与TensorRT包里面wheel包安装形式不同，这种方式是自己管理TensorRT安装，不需要提前安装TensorRT包。目前只支持Python 3.6～3.9和CUDA 11.4。

安装前的准备
```
python3 -m pip install nvidia-pyindex
```
pip install时需要额外指定```--extra-index-url https://pypi.ngc.nvidia.com```

安装TensorRT wheel包
```
python3 -m pip install --upgrade nvidia-tensorrt
```
进行验证
```
python3
>>> import tensorrt
>>> print(tensorrt.__version__)
>>> assert tensorrt.Builder(tensorrt.Logger())
```

# TensorRT生态
## basic workflow
![workflow](https://user-images.githubusercontent.com/1312389/150677789-25ed7568-b01d-4ccc-9e15-0266a61ae2c2.png)

## convert
* 使用[TF-TRT](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/compiler/tf2tensorrt)
* 使用[Torch-TensorRT](https://github.com/NVIDIA/Torch-TensorRT)
* onnx转换器转换.onnx模型
* 使用[TensorRT API](https://docs.nvidia.com/deeplearning/tensorrt/api/index.html)进行组网

## deploy
* 使用 TensorFlow

  使用 TensorFflow 模型部署即可，TensorRT不支持的OP，会fall back到TensorFlow实现。
* 使用 TRT Runtime API

  开销最小，能实现细粒度控制。对于不是原生支持的OP，需要使用plugin进行实现
* 使用 [Nvidia Triton Inference Server](https://github.com/triton-inference-server/server)
  
  能支持多种框架，包括 TensorFlow, TensorRT, PyTorch, ONNX Runtime, 或者自定义框架。

# TensorRT 基础介绍
## 创建引擎
```
Logger gLogger;
IBuilder* builder = createInferBuilder(gLogger);
nvinfer1::INetworkDefinition* network = builder->createNetworkV2(1U << static_cast<uint32_t>(NetworkDefinitionCreationFlag::kEXPLICIT_BATCH));
```
## 构建推理
```
IBuilderConfig* config = builder->createBuilderConfig();
config->setMemoryPoolLimit(1 << 20);
//设置推理精度
config->setFlag(nvinfer1::BuilderFlag::kFP16);

engine = builder->buildSerializedNetwork(*network, *config);
context = engine->createExecutionContext();
```

```
void* buffers[n];
engine->getBindingIndex(
context->enqueueV2(buffers, stream, nullptr);
```
## dynamic shape
 createNetwork()与createNetworkV2()的区别有两处，一是前者处理的维度为(C,H,W), 后者为(B,C,H,W)；二是后者支持dynamic shapes。
## plugin

 * createNetwork()
 * createNetworkV2()
## TensorRT 优化
 https://blog.csdn.net/qq_33287871/article/details/117201271
 * Weight &Activation Precision Calibration
 * Layer & Tensor Fusion
 * Kernel Auto-Tuning
 * Dynamic Tensor Memory
 * Multi-Stream Execution

# TensorRT API
 ```
 bool reshapeWeights(
     const Weights& input, int32_t const* shape, int32_t const* shapeOrder, void* data, int32_t nbDims) noexcept;
 bool reorderSubBuffers(
     void* input, int32_t const* order, int32_t num, int32_t size) noexcept;
 bool transposeSubBuffers(
     void* input, DataType type, int32_t num, int32_t height, int32_t width) noexcept;
 ```
# TensorRT 常见问题

# DLA
## DLA Supported Layers
# reference
[TensorRT](https://github.com/NVIDIA/TensorRT)

[TensorRT Developer Guide](https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/)

[TensorRT API](https://docs.nvidia.com/deeplearning/tensorrt/api/index.html)

[ONNX-TensorRT](https://github.com/onnx/onnx-tensorrt)

[Torch-TensorRT](https://github.com/NVIDIA/Torch-TensorRT)