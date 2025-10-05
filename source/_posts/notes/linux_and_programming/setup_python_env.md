---
title: "Setup Python environment"
excerpt: Linux环境下Python 3.12安装配置指南，包括从源码编译安装、使用deadsnakes PPA安装、pip配置以及设置Python 3.12为默认版本的完整步骤。
---

## linux
### Install Python 3.12 on Ubuntu 22.04
- Manually build Python 3.12 from the source code
    ```shell
    apt install -y build-essential libssl-dev zlib1g-dev libbz2-dev \
        libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
        xz-utils tk-dev libffi-dev liblzma-dev python3-openssl git

    wget https://www.python.org/ftp/python/3.12.0/Python-3.12.0.tgz
    tar -xf Python-3.12.0.tgz
    cd Python-3.12.0
    ./configure --enable-optimizations
    make -j 8
    make altinstall
    ```
- Install Python 3.12 from the deadsnakes PPA
    ```shell
    apt install software-properties-common -y
    add-apt-repository ppa:deadsnakes/ppa
    apt update -y
    ```
- pip
    ```shell
    curl -sS https://bootstrap.pypa.io/get-pip.py | python3.12
    ```
- set python3.12 default
    ```shell
    sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.12 20
    sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 20
    ```