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

# conv
| input      | filter    | output     |
| ---------- | --------- | ---------- |
| 1x32x40x80 | 16x32x3x3 | 1x16x40x80 |

int kernel_size = kernel_w * kernel_h;
int num_input = weight_data_size / kernel_size / num_output;

# cento8 epel配置aliyun源
首先安装epel配置包
yum install -y  https://mirrors.aliyun.com/epel/epel-release-latest-8.noarch.rpm
然后将 repo 配置中的地址替换为阿里云镜像站地址
sed -i 's|^#baseurl=https://download.fedoraproject.org/pub|baseurl=https://mirrors.aliyun.com|' /etc/yum.repos.d/epel*
sed -i 's|^metalink|#metalink|' /etc/yum.repos.d/epel*
