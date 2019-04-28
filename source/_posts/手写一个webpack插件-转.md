---
title: 手写一个webpack插件(转)
date: 2019-04-28 09:08:34
tags: webpack
---

[原文地址](https://segmentfault.com/a/1190000019010101)

webpack本质上是一种事件流的机制，核心是[`Tapable`][]。

webpack中最核心的负责编译的`Compiler`和负责创建`bundles`的`Compilation`都是[`Tapable`][]的实例。

## [`Tapable`][]是什么？

[`Tapable`][]暴露了很多`Hook`(钩子)类，为插件提供挂载。

```javascript
const {
  SyncHook,
  SyncBailHook,
  SyncWaterfallHook,
  SyncLoopHook,
  AsyncParallelHook,
  AsyncParallelBailHook,
  AsyncSeriesHook,
  AsyncSeriesBailHook,
  AsyncSeriesWaterfallHook
} = require('tapable');
```

### [`Tapable`][]用法

1. 新建钩子`new Hook();`
2. 使用`tap/tapAsync/tapPromise`绑定钩子
3. `call/callAsync`执行绑定事件

```javascript
// 1
const hook1 = new SyncHook(['arg1', 'arg2', 'arg3']);
// 2
hook1.tap('hook1', (arg1, arg2, arg3) => console.log(arg1, arg2, arg3));
// 3
hoo1.call(1, 2, 3); // 1, 2, 3
```

举个例子

```javascript
const { SyncHook, AsyncParallelHook } = require('tapable');

class Car {
  constructor() {
    this.hooks = {
      accelerate: new SyncHook(['newSpeed']),
      break: new SyncHook(),
      caculateRoutes: new AsyncParallelHook(['source', 'target', 'routesList'])
    };
  }
}

const myCar = new Car();

myCar.hooks.break.tap('WarningLampPlugin', () => console.log('WarningLampPlugin'));

myCar.hooks.accelerate.tap('LoggerPlugin', newSpeed => console.log(`Accelerate to ${newSpeed}`));

myCar.hooks.calculateRoutes.tapPromise('calculateRoutes tapPromise', (source, target, routesList, callback) => {
  return new Promise((resolve, rejct) => {
    setTimeout(() => {
      console.log(`tapPromise to ${source} ${target} ${routesList}`);
      resolve();
    }, 1000)
  })
});

myCar.hooks.break.call();
myCar.hooks.accelerate.call('hello');

console.time('cost');

myCar.hooks.calculateRoutes.promise('i', 'love', 'tapable').then(() => {
  console.timeEnd('cost');
}).catch(err => {
  console.error(err);
  console.timeEnd('cost');
})
```

### 进阶一下

上面学习了[`Tapable`][]的用法，但它和`webpack/plugin`有什么关系呢？

我们将刚才但代码稍作改动，拆成两个文件：`Compiler.js`、`MyPlugin.js`.

```javascript
const {
  SyncHook,
  AsyncParallelHook
} = require('tapable');

class Compiler {
  constructor(options) {
    this.hooks = {
      accelerate: new SyncHook(['newSpeed']),
      break: new SyncHook(),
      calculateRoutes: new AsyncParallelHook(['source', 'target', 'routesList'])
    };

    let plugins = options.plugins;
    if (plugins && plugins.length > 0) {
      plugins.forEach(plugin => plugin.apply(this));
    }
  }

  run() {
    console.time('cost');
    this.accelerate('hello');
    this.break();
    this.caculateRoutes('i', 'love', 'tapable');
  }

  accelerate(param) {
    this.hooks.accelerate.call(param);
  }

  break() {
    this.hooks.break.call();
  }

  calculateRoutes() {
    const args = Array.from(arguments);
    this.hooks.calculateRoutes.callAsync(...args, err=> {
      console.time('cost');
      if (err) console.error(err);
    });
  }
}

module.exports = Compiler;
```

```javascript
const Compiler = require('./Compiler');

class MyPlugin {
  constructor() {}

  apply(compiler) {
    compiler.hooks.break.tap('WarningLampPlugin', () => console.log('WarningLampPlugin'));
    compiler.hooks.accelerate.tap('LoggerPlugin', newSpeed => console.log(`Accelerating to ${newSpeed}`));
    compiler.hooks.calculateRoutes.tapAsync('calculateRoutes tapAsync', (source, target, routesList, callback) => {
      setTimeout(() => {
        console.log(`tapAsync to ${source} ${target} ${routesList}`);
        callback();
      }, 2000);
    });
  }
}

const myPlugin = new MyPlugin();
const options = {
  plugins: [myPlugin]
}
let compiler = new Compiler(options);
compiler.run();
```

## Plugin基础

webpack通过`plugin`机制让其更加灵活，以适应各种应用场景。在webpack运行的生命周期中会广播出许多事件，`plugin`可以监听这些事件，在合适的时机通过webpack提供的API改变输出结果。

一个最基础的`plugin`的代码是这样的：

```javascript
class PluginDemo {
  constructor() {}

  apply(compiler) {
    compiler.hooks.compilation.tap('PluginDemo', compilation => {});
  }
}

module.exports = PluginDemo;
```

在使用这个`plugin`时，相关代码如下：

```javascript
const PluginDemo = require('./PluginDemo');

module.exports = {
  plugins: [
    new PluginDemo(options);
  ]
}
```

### `Compiler`和`Compilation`

在开发`plugin`时最常用的两个对象就是`Compiler`和`Compilation`，它们是`Plugin`和webpack之间的桥梁。

#### `Compiler`

`Compiler`对象包含了webpack环境所有的配置信息，包含`options`，`loaders`，`plugins`等，这个对象在webpack启动的时候被实例化，它在全局是唯一的。

#### `Compilation`

`Compilation`对象包含了当前的模块资源、编译生成资源、变化的文件等。当webpack以开发模式运行时，每当检测到一个文件变化，一次新的`Compilation`将被创建。`Compilation`对象也提供了很多事件回调供插件做扩展。通过`Compilation`对象也可以读取到`Compiler`对象。

它们的区别在于：`Compiler`代表了整个webpack从启动到关闭的生命周期，而`Compilation`只是代表了一次新的编译。

## 常用API

## [Writing a Plugin][]

[`Tapable`]: https://github.com/webpack/tapable
[Writing a Plugin]: https://webpack.js.org/contribute/writing-a-plugin/#creating-a-plugin
