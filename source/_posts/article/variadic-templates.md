---
title: 可变模版参数
date: 2021-04-04 14:06:42
tags: c++
---

# c++11 可变模版参数
```
template <class... T>
void f(T... args);
```
* 递归函数展开参数包
```
#include <iostream>

template <class... T>
void f(T... args)
{    
  std::cout << sizeof...(args) << std::endl;
}

void func() {}
template <class T, class... Args>
void func(T first, Args... remain) {
  std::cout << first << " ";
  if (sizeof...(remain) == 0) return;
  func(remain...);
}

int main() {
  func(2, 3, 9);
  return 0;
}
```
* 逗号表达式展开参数包