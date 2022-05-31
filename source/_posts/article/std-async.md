---
title: 'std::async、std::future'
date: 2021-04-05 14:22:20
tags:
---

## std::async 用法
```
template<class Fn, class... Args>
future<typename result_of<Fn(Args...)>::type> async(launch policy, Fn&& fn, Args&&...args);
```
* std::launch::async
  系统默认，调用时创建新线程, 
* std::launch::deferred
  延迟到std::future调用wait()或者get()时才执行，主线程调用，不创建新线程

std::async 封装
```
template <typename F, typename... Args>
auto really_async(F&& f, Args&&... args)
-> std::future<typename std::result_of<F(Args...)>::type>
{
    using RetType = typename std::result_of<F(Args...)>::type;
    auto func = std::bind(std::forward<F>(f), std::forward<Args>(args)...);
    std::packaged_task<RetType()> task(std::move(func));
    auto fut = task.get_future();
    std::thread trd(std::move(task));
    trd.detach();
    return fut;
}
```

## std::future 用法

### std::future_status 三种状态
* deferred
  异步操作待开始
* ready
  异步操作完成
* timeout
  异步操作超时

## std::promise
## std::packaged_task