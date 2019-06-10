---
title: tsconfig.json(译)
date: 2019-05-29 21:42:12
tags:
- TypeScript
- tsconfig.json
---

[原文地址](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)

`tsconfig.json`文件通常放在工程根目录下，用来定义编译的配置。

## 使用

* 不提供任何输入文件的配置，直接调用`tsc`，`compiler`会找`tsconfig.json`，从当前文件夹开始，找不到就往父目录找。
* 不提供任何输入文件的配置，使用`—project`或者`-p`，调用`tsc`，指定一个包含`tsconfig.json`文件的文件夹路径，或者一个包含配置信息的合法.json文件的路径。

如果指定了输入文件，那么`tsconfig.json`文件会被忽略。

## 示例

```json
// 使用 file
{
  "compilerOptions": {
    "module": "commonjs",
    "noImplicitAny": true,
    "removeComments": true,
    "preserveConstEnums": true,
    "sourceMap": true
  },
  "files": [
    "core.ts",
    "sys.ts",
    "types.ts",
    "scanner.ts",
    "parser.ts",
    "utilities.ts",
    "binder.ts",
    "checker.ts"
  ]
}

// 使用 include 和 exclude
{
  "compilerOptions": {
    "module": "system",
    "noImplicitAny": true,
    "removeComments": true,
    "preserveConstEnums": true,
    "outFile": "../../built/local/tsc.js",
    "sourceMap": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "**/*.spec.ts"
  ]
}
```

## 细节

`compilerOptions`可以忽略，`compiler`会使用默认配置。[`Compiler Options`][]

`files`接收一组相对/绝对路径。`include`/`exclude`接受一组glob-like file pattern。支持的glob通配符如下：

* `*`匹配0或多个字符（除了文件夹分隔符）
* `?`匹配任意1个字符（除了文件夹分隔符）
* `**/`递归匹配任意子目录

如果某glob pattern只包含`*`/`.*`，那么默认包含的文件为所有扩展名为`.ts`/`.tsx`/`.d.ts`的文件。（如果`allowJs`为`true`，那么扩展名为`.js`/`.jsx`的文件也被包含在内）

如果`files`/`include`都没有指定值，那么`compiler`默认包含文件夹/子文件夹下的所有`TypeScript`文件。（扩展名为`.ts`/`.tsx`/`.d.ts`的文件。如果`allowJs`为`true`，那么扩展名为`.js`/`.jsx`的文件也被包含在内）

`exclude`可以过滤掉`include`指定的文件，但不能过滤掉`files`指定的文件。如果`exclude`没有指定值，默认包含`node_modules`/`bower_components`/`jspm_packages`/`<outDir>`。

被包含的文件引用的文件，也被包含在内，不会被`exclude`过滤掉，除非引用它的文件被过滤掉了。

`compiler`会过滤掉那些可能成为输出文件的输入文件。例如，同一目录下同时有`[name].ts`/`[name].d.ts`/`[name].js`，那么`[name].d.ts`/`[name].js`会被忽略。

`tsconfig.json`可以是一个空文件，`compiler`会使用默认配置。

## `@types`，`typeRoots`和`types`

默认情况下，只有`@types`目录下的包会被包含在`compilation`里。例如，`./node_modules/@types`/`../node_modules/@types`/`../../node_modules/@types`。

如果指定了`typeRoots`的值，那么只有`typeRoots`目录下的包会被包含进来。

例如：

```json
{
  "compilerOptions": {
    "typeRoots": ["./typings"]
  }
}
```

只有`./typings`目录下的包会被包含进来，`./node_modules/@types`目录下的包不会被包含进来。

如果指定了`types`的值，那么只有列举出来的包会被包含进来。

```json
{
  "compilerOptions": {
    "types": ["node", "lodash", "express"]
  }
}
```

只有`./node_modules/@types/node`/`./node_modules/@types/lodash`/`./node_modules/@types/express`会被包含进来，其它`./node_modules/@types/*`目录下的包不会被包含进来。

指定`types`值为`[]`，将会自动过滤所有`@types`包。

## 使用`extends`继承配置文件

例如：

configs/base.json

```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

tsconfig.json

```json
{
  "extends": "./configs/base",
  "files": [
    "main.ts",
    "supplemental.ts"
  ]
}
```

tsconfig.nostrictnull.json

```json
{
  "extends": "./configs/base",
  "compilerOptions": {
    "strictNullChecks": false
  }
}
```

## compileOnSave

告诉IDE，在保存文件的时候使用给定的`tsconfig.json`处理所有的文件。

```json
{
  "compileOnSave": true,
  "compilerOptions": {
    "noImplicitAny": true
  }
}
```

## [`Schema`][]

[`Compiler Options`]: https://www.typescriptlang.org/docs/handbook/compiler-options.html
[`Schema`]: http://json.schemastore.org/tsconfig
