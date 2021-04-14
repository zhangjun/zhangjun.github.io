---
title: models
date: 2021-03-23 15:14:00
tags:
---
介绍paddle、ncnn、tnn使用

# paddle
## x2paddle
x2paddle --framework=onnx --model=onnx_model.onnx --save_dir=mobilenet

## paddle2onnx
paddle2onnx --model_dir paddle_model  --save_file onnx_file --opset_version 10 --enable_onnx_checker True
paddle2onnx --model_dir paddle_model  --model_filename model_filename --params_filename params_filename --save_file onnx_file --opset_version 10 --enable_onnx_checker True
