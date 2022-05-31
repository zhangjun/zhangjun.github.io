<!--
 * @Author: Zhang Jun ewalker@live.cn
 * @Date: 2022-05-31 21:53:32
 * @LastEditors: Zhang Jun ewalker@live.cn
 * @LastEditTime: 2022-05-31 22:10:33
 * @FilePath: /undefined/Users/apple/Downloads/zhangjun/github/zhangjun.github.io/source/_posts/article/paddle-model.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
---
title: export paddle model
date: 2022-05-31 13:53:32
tags:
---

# 导出模型
以[PaddleClas](https://github.com/PaddlePaddle/PaddleClas)为例。

- 下载预训练模型
  进入https://github.com/PaddlePaddle/PaddleClas/blob/2.3.0/docs/en/ImageNet_models_en.md。
  下载MobileNetV1预训练模型。
  ```
  wget https://paddle-imagenet-models-name.bj.bcebos.com/dygraph/legendary_models/MobileNetV1_pretrained.pdparams
  ```
- 导出模型
  ```
  python tools/export_model.py -c ./ppcls/configs/ImageNet/MobileNetV1/MobileNetV1.yaml -o Global.pretrained_model=clas_model/MobileNetV1/MobileNetV1_pretrained
  ```
- da