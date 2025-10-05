---
title: "Setup CUDA environment"
excerpt: CUDA开发环境配置指南，包括nvcc编译器安装、NVIDIA容器镜像使用、网络仓库安装方法，以及nsys性能分析工具的安装配置步骤。
---

## nvcc
```shell
distro=ubuntu2204
arch=x86_64
wget https://developer.download.nvidia.com/compute/cuda/repos/<distro>/<arch>/cuda-keyring_1.1-1_all.deb
# or curl -fsSLO https://developer.download.nvidia.com/compute/cuda/repos/<distro>/<arch>/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb
```
- [nvidia container-images](https://gitlab.com/nvidia/container-images/cuda/-/tree/master/dist)
- [network-repo-installation-for-ubuntu](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#network-repo-installation-for-ubuntu)

## nsys
```shell
apt update
apt install -y --no-install-recommends gnupg
echo "deb http://developer.download.nvidia.com/devtools/repos/ubuntu$(source /etc/lsb-release; echo "$DISTRIB_RELEASE" | tr -d .)/$(dpkg --print-architecture) /" | tee /etc/apt/sources.list.d/nvidia-devtools.list
apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
apt update
apt install nsight-systems-cli
```