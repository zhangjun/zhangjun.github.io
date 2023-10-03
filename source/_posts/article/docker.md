---
title: docker
date: 2021-12-09 06:33:23
tags:
---
# docker build
https://docs.docker.com/engine/reference/builder/

## install docker

```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
# ubuntu docker

 https://askubuntu.com/questions/1140183/install-gcc-9-on-ubuntu-18-04
```
FROM ubuntu:16.04

LABEL com.zhangjun.image.authors="ewalker.zj@gmail.com"

ENV TZ "Asia/Shanghai"

RUN apt update && \
    apt -qqy install software-properties-common && \
    add-apt-repository -y  ppa:ubuntu-toolchain-r/test && \
    add-apt-repository -y ppa:deadsnakes/ppa && \
    apt update && \
    apt -qqy install gcc-9 g++-9 && \
    apt -qqy install python3.7 && \
    update-alternatives --install /usr/bin/python python /usr/bin/python3.7 10 && \
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 40 --slave /usr/bin/g++ g++ /usr/bin/g++-9 && \
    wget https://bootstrap.pypa.io/get-pip.py && \
    python3.7 get-pip.py && \
    python3.7 -m pip install pre-commit && \
    apt-get -qqy clean && \
    rm -rf get-pip.py && rm -rf /var/lib/apt/lists/*

#    update-alternatives --config gcc
#    update-alternatives --config python
```