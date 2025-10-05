---
title: ssh setup
excerpt: SSH配置和Git使用指南，包括GitHub SSH密钥生成、SSH配置文件设置、文件权限配置，以及Git常用命令如深度克隆特定分支等实用技巧。
---

## github ssh配置

### 生成key
```
ssh-keygen -t rsa -f ~/.ssh/baidu_id_rsa
```
### 配置～/.ssh/config文件
```
#GitHub
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
```
~/.ssh/config 文件权限必须为644

## git 常用命令
git clone --depth 1 --branch v5.0.8 --no-checkout https://github.com/emqx/emqx.git