---
title: 常用命令
date: 2021-03-11 08:50:47
tags:
---
记录linux常用命令
# shell
* 目录下文件中指定字符串替换
  将当前目录包括子目录的文件中`printf`字符串替换为`print`
  ```
  sed -i "s/printf/print/g" `grep printf -rl ./`
  ```