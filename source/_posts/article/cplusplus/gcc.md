---
title: gcc
date: 2021-04-14 03:51:57
tags:
---

# content

## RVO(Return Value Optimization) 
## NRVO(Named Return Value Optimization)

## code snipts
```
#include <algorithm>
#include <cctype>

std::string lower(const std::string &data) {
  std::string result = data;
  std::transform(result.begin(), result.end(), result.begin(),
    [](unsigned char c){ return std::tolower(c); });
  return result;
}
```