---
title: 深入理解TypeScript(一)
date: 2019-04-19 19:58:07
tags:
---
[原文链接](https://jkchao.github.io/typescript-book-chinese/)
## 编译上下文
用来给文件分组，告诉TS哪些文件是有效的，哪些是无效的。
包含了有哪些编译选项正在使用。
定义这种逻辑分组，一个比较好的方式是使用`tsconfig.json`文件。
#### tsconfig.json
在项目根目录下创建一个空json文件，通过这种方式，TS会将此目录下的所有`.ts`文件作为编译上下文的一部分，还会包含一部分默认的编译选项。
#### 编译选项
通过`compilerOptions`来定制：
```json
{
  "compilerOptions": {

    /* 基本选项 */
    "target": "es5",                       // 指定 ECMAScript 目标版本: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', or 'ESNEXT'
    "module": "commonjs",                  // 指定使用模块: 'commonjs', 'amd', 'system', 'umd' or 'es2015'
    "lib": [],                             // 指定要包含在编译中的库文件
    "allowJs": true,                       // 允许编译 javascript 文件
    "checkJs": true,                       // 报告 javascript 文件中的错误
    "jsx": "preserve",                     // 指定 jsx 代码的生成: 'preserve', 'react-native', or 'react'
    "declaration": true,                   // 生成相应的 '.d.ts' 文件
    "sourceMap": true,                     // 生成相应的 '.map' 文件
    "outFile": "./",                       // 将输出文件合并为一个文件
    "outDir": "./",                        // 指定输出目录
    "rootDir": "./",                       // 用来控制输出目录结构 --outDir.
    "removeComments": true,                // 删除编译后的所有的注释
    "noEmit": true,                        // 不生成输出文件
    "importHelpers": true,                 // 从 tslib 导入辅助工具函数
    "isolatedModules": true,               // 将每个文件做为单独的模块 （与 'ts.transpileModule' 类似）.

    /* 严格的类型检查选项 */
    "strict": true,                        // 启用所有严格类型检查选项
    "noImplicitAny": true,                 // 在表达式和声明上有隐含的 any类型时报错
    "strictNullChecks": true,              // 启用严格的 null 检查
    "noImplicitThis": true,                // 当 this 表达式值为 any 类型的时候，生成一个错误
    "alwaysStrict": true,                  // 以严格模式检查每个模块，并在每个文件里加入 'use strict'

    /* 额外的检查 */
    "noUnusedLocals": true,                // 有未使用的变量时，抛出错误
    "noUnusedParameters": true,            // 有未使用的参数时，抛出错误
    "noImplicitReturns": true,             // 并不是所有函数里的代码都有返回值时，抛出错误
    "noFallthroughCasesInSwitch": true,    // 报告 switch 语句的 fallthrough 错误。（即，不允许 switch 的 case 语句贯穿）

    /* 模块解析选项 */
    "moduleResolution": "node",            // 选择模块解析策略： 'node' (Node.js) or 'classic' (TypeScript pre-1.6)
    "baseUrl": "./",                       // 用于解析非相对模块名称的基目录
    "paths": {},                           // 模块名到基于 baseUrl 的路径映射的列表
    "rootDirs": [],                        // 根文件夹列表，其组合内容表示项目运行时的结构内容
    "typeRoots": [],                       // 包含类型声明的文件列表
    "types": [],                           // 需要包含的类型声明文件名列表
    "allowSyntheticDefaultImports": true,  // 允许从没有设置默认导出的模块中默认导入。

    /* Source Map Options */
    "sourceRoot": "./",                    // 指定调试器应该找到 TypeScript 文件而不是源文件的位置
    "mapRoot": "./",                       // 指定调试器应该找到映射文件而不是生成文件的位置
    "inlineSourceMap": true,               // 生成单个 soucemaps 文件，而不是将 sourcemaps 生成不同的文件
    "inlineSources": true,                 // 将代码与 sourcemaps 生成到一个文件中，要求同时设置了 --inlineSourceMap 或 --sourceMap 属性

    /* 其他选项 */
    "experimentalDecorators": true,        // 启用装饰器
    "emitDecoratorMetadata": true          // 为装饰器提供元数据的支持
  }
}
```
#### 编译
好的IDE支持即时编译，也可以从命令行手动运行编译器。
* 运行tsc，会在当前目录或者父级目录寻找`tsconfig.json`文件
* 运行`tsc -p ./path-to-project-directory`。这个路径可以是绝对路径，也可以是相对路径。

使用`tsc -w`启用TS编译器的观测模式，在检测到文件改动后，将重新编译。
#### 指定文件
也可以显式指定需要编译的文件：
```json
{
  "files": [
    "./some/file.ts"
  ]
}
```
或者使用`include`和`exclude`选项来指定需要包含的文件和排除的文件：
```json
{
  "include": [
    "./folder"
  ],
  "exclude": [
    "./folder/**/*.spec.ts",
    "./folder/someSubFolder"
  ]
}
```
## 声明空间
两种声明空间：类型声明空间和变量声明空间。
#### 类型声明空间
当作类型注解的内容：
```typescript
class Foo {}
interface Bar {}
type Bas = {};
```
这里可以将`Foo`，`Bar`，`Bas`当作类型注解使用：
```typescript
let foo: Foo;
let bar: Bar;
let bas: Bas;
```
注意，下面尽管定义了`interface Bar`，但不能将它作为变量使用，因为它没有定义在变量声明空间中：
```typescript
interface Bar {}
const bar = Bar; // Error: "cannot find name 'Bar'"
```
#### 变量声明空间
变量声明空间包括可用作变量的内容，在前面`class Foo`提供了类型`Foo`到类型声明空间，同时提供了一个变量`Foo`到变量声明空间：
```typescript
class Foo {}
const someVar = Foo;
const someOtherVar = 123;
```
