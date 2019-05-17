---
title: tasks-microtasks-queues-and-schedules
date: 2019-05-17 16:03:12
tags:
- macrotasks and microtasks
---

[article](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

每个线程都有自己的`event loop`，因此每个`web worker`都有自己的`event loop`。

相同域名的窗口共享一个`event loop`，以实现窗口间的同步通信。
