---
title: notes
date: 2021-03-12 05:24:19
tags:
---
一些记录

# ccache和distcc
```
export USE_CCACHE=1
export CCACHE_DIR=/home/xx/tools/.ccache
ccache -M 50G
ccache -s
ccache -C
```
```
cmake中使用ccache的最加方案：cmake > 3.5，命令行上-DCMAKE_CXX_COMPILER_LAUNCHER=ccache配置文件
find_program(CCACHE_FOUND ccache)
    if(CCACHE_FOUND)  
        set(CMAKE_CXX_COMPILER_LAUNCHER ccache)
    endif()
```