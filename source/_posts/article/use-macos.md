---
title: macOS tips
date: 2021-12-09 06:31:52
tags:
---

# macOS python load Framework
https://docs.python.org/3/library/ctypes.html
```
from ctypes.util import find_library
from ctypes import cdll
import os

print(os.name)
metal_library = find_library("Metal")
core_graphics_library = find_library("CoreGraphics")
mps_library = find_library("MetalPerformanceShaders")

print(cdll.LoadLibrary(metal_library))
print(cdll.LoadLibrary(core_graphics_library))
print(cdll.LoadLibrary(mps_library))
```