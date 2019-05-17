---
title: 浏览器和Node.js不同的事件循环（EventLoop）
date: 2019-05-17 15:24:49
tags:
- 事件循环
- Event Loop
---

[原文地址](https://segmentfault.com/a/1190000013660033?utm_source=channel-hottest)

### 浏览器环境

js执行为单线程（不考虑web worker），所有代码皆在执行线程调用栈完成执行。**当执行线程任务清空后才会去轮询取任务队列中任务**。
