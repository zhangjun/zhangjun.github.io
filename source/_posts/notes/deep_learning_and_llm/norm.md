---
title: norm
tags: [Normalization, Deep Learning, PyTorch]
excerpt: 深度学习归一化方法详解，包括Batch Norm、Layer Norm、Instance Norm、Group Norm四种归一化技术的原理、实现方法和PyTorch代码示例，帮助理解不同归一化策略的应用场景。
---

## 归一化方法

![](https://img-blog.csdnimg.cn/20210319221042880.png)

### Batch Norm
Batch Norm在通道维度进行归一化，最后得到C个统计量u，δ。假设输入特征为[N, H, W, C]，在C的每个维度上对[N, H, W]计算其均值、方差，用于该维度上的归一化操作。
```python
import numpy as np
import torch
import torch.nn as nn
from einops import rearrange, repeat, reduce

image = [np.random.randn(30, 40, 3) for _ in range(16)]
image = rearrange(image, 'b h w c -> b h w c')
# print(rearrange(image, 'b h w c -> b h w c').shape)

image_ = rearrange(image, 'b h w c -> (b h w) c')
mean = rearrange(image_.mean(axis=0), 'c -> 1 1 1 c')
std = rearrange(image_.std(axis=0), 'c -> 1 1 1 c')

y_ =  (image - mean)/std

b, h, w, c = image.shape
bn = nn.BatchNorm2d(c, eps=1e-10, affine=False, track_running_stats=False)
y = bn(torch.from_numpy(image))

print('diff={}\n'.format(torch.abs(y - y_).max()))

```
### Layer Norm
Layer Norm以样本为单位计算统计量，因此最后会得到N个u，δ。假设输入特征为[N, H, W, C]，在N的每个维度上对[H, W，C]计算其均值、方差，用于该维度上的归一化操作。
```python
import numpy as np
import torch
import torch.nn as nn
from einops import rearrange, repeat, reduce

x = torch.randn((6, 3, 20, 20))
b, c, h, w = x.shape

layer_norm = nn.LayerNorm([c, h, w], eps=1e-12, elementwise_affine=False)
y = layer_norm(x)

x_ = rearrange(x, 'b c h w -> (h w c) b')
mean = rearrange(x_.mean(axis=0), 'b -> b 1 1 1')
std = rearrange(x_.std(axis=0), 'b -> b 1 1 1')

y_ =  (x - mean)/std

print('diff={}\n'.format(torch.abs(y - y_).max()))

```
### Instance Norm
```python
import numpy as np
import torch
import torch.nn as nn
from einops import rearrange, repeat, reduce

x = torch.randn((6, 3, 20, 20))
b, c, h, w = x.shape

instance_norm = nn.InstanceNorm2d(c, eps=1e-12, affine=False, track_running_stats=False)
y = instance_norm(x)

x_ = rearrange(x, 'b c h w -> b c (h w)')
# mean = rearrange(x_.mean(axis=2), 'b c -> b c 1 1')
# std = rearrange(x_.std(axis=2), 'b c -> b c 1 1')
mean = rearrange(x_.mean(dim=2), 'b c -> b c 1 1')
std = rearrange(x_.std(dim=2), 'b c -> b c 1 1')

y_ =  (x - mean)/std

print('diff={}\n'.format(torch.abs(y - y_).max()))

```
### Group Norm
```python
import numpy as np
import torch
import torch.nn as nn
from einops import rearrange, repeat, reduce

x = torch.randn((6, 6, 20, 20))
b, c, h, w = x.shape
group_num = 3
n = 2

group_norm = nn.GroupNorm(group_num, c, eps=1e-12, affine=False)
y = group_norm(x)

x_ = rearrange(x, 'b (g n) h w -> b g (n h w)', g = group_num) # [6, 3, 2*20*20]
print(x_.shape)
mean = rearrange(x_.mean(dim=2), 'b g -> b g 1')  # [6, 3, 1]
std = rearrange(x_.std(dim=2), 'b g -> b g 1')

y_ =  (x_ - mean)/std
y_ = rearrange(y_, 'b g (n h w) -> b (g n) h w', g = group_num, h = h, w = w)

print('diff={}\n'.format(torch.abs(y - y_).max()))

```