---
title: Modern Cpp
date: 2022-01-09 07:03:59
tags:
---

# cpp11

## type traits
* std::integral_constant

    wrap a static constant of specified type. Defined in <type_traits>
    ```
    template<class T, T v>
    struct integral_constant;
    ``` 
