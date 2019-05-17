---
title: Javascript引擎是如何工作的？从调用栈到Promise你需要知道的一切
date: 2019-05-17 13:58:02
tags:
- JavaScript
- V8引擎
---

### JavaScript引擎和全局内存

**全局内存（也称为堆）**是JavaScript引擎用来保存变量和函数声明的区域。在浏览器或在Node.js中，有许多预定义的函数和变量，被称为全局。全局内存将比我们的代码所占用的空间多。

### 全局执行上下文和调用栈

每个JavaScript引擎都有一个**基本组件，称为调用栈**。

**当调用`foo()`时，引擎会将该函数压入调用堆栈中**。

引擎同时还分配了**全局执行上下文**，这是JavaScript代码运行的全局环境。

**对于嵌套函数中的每个嵌套函数，引擎都会创建本地执行上下文**。

### 单线程的JavaScript

**当浏览器加载某些JavaScript代码时，引擎会逐行读取**并执行一下步骤：

* 使用**变量和函数声明**填充**全局内存（堆）**
* 将每个函数调用送到**调用栈**
* 创建一个**全局执行上下文**
* （如果有内部变量或嵌套函数）创建**本地执行上下文**

### 异步JavaScript，回调队列和事件循环

`setTimeout`是浏览器提供的API，该函数由浏览器直接运行（它会暂时出现在调用栈中，但会立即删除）。当时间过期后，浏览器接受我们传入的回调函数并将其移动到**回调队列**。

**每个异步函数在被送入调用栈之前，必须通过回调队列**。

由事件循环（Event Loop）组件，推动函数进入调用栈。事件循环只做一件事，**检查调用栈**是否为空。

![事件队列](https://image-static.segmentfault.com/317/571/3175718811-5cdd42f08d082_articlex)

*注意：*浏览器API、回调队列、事件循环是异步JavaScript的支柱。

### Promise

* `Promise.all`: 当任何一个Promise rejected时，Promise.all就会rejects
* `Promise.race`: 有一个Promise resolve/reject后立即resolve/reject
* `Promise.any`: 表明任何Promise是否fullfilled，即使其中有的Promise已经rejected
* `Promise.allSettled`: 检查是否全部Promise都结束

### 错误处理

```javascript
async function getData() {
  try {
    if (true) {
      throw new Error('error happens');
    }
  } catch (err) {
    console.log('error catched in the catch block');
  }
}

getData().catch(err => console.log('error catched in the promise .catch'));

// error catched in the catch block
```

使用`async`定义的函数会包裹在Promise里，抛出的错误在内部的`catch`块被捕获，不会再传播到`.catch`里。
