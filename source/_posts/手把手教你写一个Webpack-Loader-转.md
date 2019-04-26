---
title: 手把手教你写一个Webpack Loader(转)
date: 2019-04-26 11:42:15
tags:
---

[原文链接](https://segmentfault.com/a/1190000018980814)
[官方参考writing-a-loader](https://webpack.js.org/contribute/writing-a-loader)

首先我们来看一下为什么需要`loader`，以及它能干什么?
webpack 只能理解 JavaScript 和 JSON 文件。`loader` 让 webpack 能够去处理其他类型的文件，并将它们转换为有效模块，以供应用程序使用，以及被添加到依赖图中。

本质上来说，`loader` 就是一个 node 模块，这很符合 webpack 中「万物皆模块」的思路。既然是 node 模块，那就一定会导出点什么。在 webpack 的定义中，`loader` 导出一个函数，`loader` 会在转换源模块resource的时候调用该函数。在这个函数内部，我们可以通过传入 `this` 上下文给 `Loader API` 来使用它们。最终装换成可以直接引用的模块。

## `xml-loader`实现

前面我们已经知道，由于 Webpack 是运行在 Node.js 之上的，一个 `Loader` 其实就是一个 Node.js 模块，这个模块需要导出一个函数。 这个导出的函数的工作就是获得处理前的原内容，对原内容执行处理后，返回处理后的内容。
一个简单的`loader`源码如下

```javascript
module.exports = function(source) {
  // source 为 compiler 传递给 `Loader` 的一个文件的原内容
  // 该函数需要返回处理后的内容，这里简单起见，直接把原内容返回了，相当于该 `Loader` 没有做任何转换
  return source;
};
```

由于 `Loader` 运行在 Node.js 中，你可以调用任何 Node.js 自带的 API，或者安装第三方模块进行调用：

```javascript
const xml2js = require('xml2js');
const parser = new xml2js.Parser();

module.exports =  function(source) {
  this.cacheable && this.cacheable();
  const self = this;
  parser.parseString(source, function (err, result) {
    self.callback(err, !err && "module.exports = " + JSON.stringify(result));
  });
};
```

这里我们事简单实现一个`xml-loader`;

注意：如果是处理顺序排在最后一个的 `loader`，那么它的返回值将最终交给 webpack 的 `require`，换句话说，它一定是一段可执行的 JS 脚本 （用字符串来存储），更准确来说，是一个 node 模块的 JS 脚本，所以我们需要用`module.exports =`导出。
整个过程相当于这个 `loader` 把源文件

    // 这里是 source 模块

转化为

```javascript
// example.js
module.exports = '这里是 source 模块';
```

然后交给 require 调用方：

```javascript
// applySomeModule.js
var source = require('example.js'); 
console.log(source); // 这里是 source 模块
```

写完后我们要怎么在本地验证呢？下面我们来写个简单的demo进行验证。

## 验证

首先我们创建一个根目录`xml-loader`，此目录下 `npm init -y`生成默认的`package.json`文件 ,在文件中配置打包命令

```json
"scripts": {
    "dev": "webpack-dev-server"
  },
```

之后`npm i -D webpack webpack-cli`,安装完webpack，在根目录 创建配置文件`webpack.config.js`

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.xml$/,
        use: ['xml-loader'],
      }
    ]
  },
  resolveLoader: {
    modules: [path.join(__dirname, '/src/loader')]
  },
  devServer: {
    contentBase: './dist',
    overlay: {
      warnings: true,
      errors: true
    },
    open: true
  }
}
```

在根目录创建一个`src`目录，里面创建`index.js`,

```javascript
import data from './foo.xml';

function component() {
  var element = document.createElement('div');
  element.innerHTML = data.note.body;
  element.classList.add('header');
  console.log(data);
  return element;
}

document.body.appendChild(component());
```

同时还有一个foo.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<note>
    <to>Mary</to>
    <from>John</from>
    <heading>Reminder  dd</heading>
    <body>Call Cindy on Tuesday dd</body>
</note>
```

最后把上面的`xml-loader`放到`src/loader`文件夹下。
完整的[demo源码](https://github.com/xiangxingchen/blog/tree/master/src/webpack/demo06)请看
最终我们的运行效果如下图

![图片描述](https://image-static.segmentfault.com/416/401/4164017475-5cc06632a147b_articlex)

至此一个简单的`webpack loader`就实现完成了。当然最终使用你可以发布到npm上。

## 一些议论知识补充

### 获得`Loader`的`options`

当我们配置loader时我们经常会看到有这样的配置

```javascript
ules: [{
    test: /\.html$/,
    use: [ {
      loader: 'html-loader',
      options: {
        minimize: true
      }
    }],
  }]
```

那么我们在`loader`中怎么获取这写配置信息呢？答案是`loader-utils`。这个由webpack提供的工具。下面我们来看下使用方法

```javascript
const loaderUtils = require('loader-utils');
module.exports = function(source) {
  // 获取到用户给当前 Loader 传入的 options
  const options = loaderUtils.getOptions(this);
  return source;
};
```

没错就是这么简单。

### 加载本地`Loader`

#### `path.resolve`

可以简单通过在 `rule` 对象设置 `path.resolve` 指向这个本地文件

```javascript
{
  test: /\.js$/
  use: [
    {
      loader: path.resolve('path/to/loader.js'),
      options: {/* ... */}
    }
  ]
}
```

#### ResolveLoader

这个就是上面我用到的方法。`ResolveLoader` 用于配置 Webpack 如何寻找 `Loader`。 默认情况下只会去 `node_modules` 目录下寻找，为了让 Webpack 加载放在本地项目中的 `Loader` 需要修改 `resolveLoader.modules`。
假如本地的 `Loader` 在项目目录中的 `./loaders/loader-name` 中，则需要如下配置：

```javascript
module.exports = {
  resolveLoader:{
    // 去哪些目录下寻找 Loader，有先后顺序之分
    modules: ['node_modules','./loaders/'],
  }
}
```

加上以上配置后， Webpack 会先去 `node_modules` 项目下寻找 `Loader`，如果找不到，会再去 `./loaders/` 目录下寻找。

#### [npm link][]

[npm link][] 专门用于开发和调试本地 `npm` 模块，能做到在不发布模块的情况下，把本地的一个正在开发的模块的源码链接到项目的 `node_modules` 目录下，让项目可以直接使用本地的 `npm` 模块。 由于是通过软链接的方式实现的，编辑了本地的 `npm` 模块代码，在项目中也能使用到编辑后的代码。

完成 [npm link][] 的步骤如下：

* 确保正在开发的本地 `npm` 模块（也就是正在开发的 `Loader`）的 `package.json` 已经正确配置好；
* 在本地 `npm` 模块根目录下执行 [npm link][]，把本地模块注册到全局；
* 在项目根目录下执行 [npm link][] `loader-name`，把第2步注册到全局的本地 `npm` 模块链接到项目的 `node_moduels` 下，其中的 `loader-name` 是指在第1步中的`package.json` 文件中配置的模块名称。

链接好 `Loader` 到项目后你就可以像使用一个真正的 `npm` 模块一样使用本地的 `Loader` 了。([npm link][]不是很熟，复制被人的)

### 缓存加速

在有些情况下，有些转换操作需要大量计算非常耗时，如果每次构建都重新执行重复的转换操作，构建将会变得非常缓慢。 为此，Webpack 会默认缓存所有 `Loader` 的处理结果，也就是说在需要被处理的文件或者其依赖的文件没有发生变化时， 是不会重新调用对应的 `Loader` 去执行转换操作的。

如果你想让 Webpack 不缓存该 `Loader` 的处理结果，可以这样：

```javascript
module.exports = function(source) {
  // 关闭该 Loader 的缓存功能
  this.cacheable(false);
  return source;
};
```

### 处理二进制数据

在默认的情况下，Webpack 传给 `Loader` 的原内容都是 UTF-8 格式编码的字符串。 但有些场景下 `Loader` 不是处理文本文件，而是处理二进制文件，例如 `file-loader`，就需要 Webpack 给 `Loader` 传入二进制格式的数据。 为此，你需要这样编写 `Loader`：

```javascript
module.exports = function(source) {
    // 在 exports.raw === true 时，Webpack 传给 Loader 的 source 是 Buffer 类型的
    source instanceof Buffer === true;
    // Loader 返回的类型也可以是 Buffer 类型的
    // 在 exports.raw !== true 时，Loader 也可以返回 Buffer 类型的结果
    return source;
};
// 通过 exports.raw 属性告诉 Webpack 该 Loader 是否需要二进制数据 
module.exports.raw = true;
```

以上代码中最关键的代码是最后一行 `module.exports.raw = true;`，没有该行 `Loader` 只能拿到字符串。

### 同步与异步

`Loader` 有同步和异步之分，上面介绍的 `Loader` 都是同步的 `Loader`，因为它们的转换流程都是同步的，转换完成后再返回结果。 但在有些场景下转换的步骤只能是异步完成的，例如你需要通过网络请求才能得出结果，如果采用同步的方式网络请求就会阻塞整个构建，导致构建非常缓慢。

在转换步骤是异步时，你可以这样：

```javascript
module.exports = function(source) {
    // 告诉 Webpack 本次转换是异步的，Loader 会在 callback 中回调结果
    var callback = this.async();
    someAsyncOperation(source, function(err, result, sourceMaps, ast) {
        // 通过 callback 返回异步执行后的结果
        callback(err, result, sourceMaps, ast);
    });
};
```

[npm link]: https://docs.npmjs.com/cli/link
