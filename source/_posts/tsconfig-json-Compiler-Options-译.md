---
title: tsconfig.json Compiler Options(译)
date: 2019-05-29 22:24:05
tags:
- TypeScript
- tsconfig.json
---

[原文地址](https://www.typescriptlang.org/docs/handbook/compiler-options.html)

Option   | Type  | Default | Description        
---------|-------|---------|--------------------
--allowJs                      | boolean |                    false                  | 允许编译Javascript文件
--allowSyntheticDefaultImports | boolean | module === 'system' 或者 --esModuleInterop | 允许在没有default export的时候从modules中default import。这不影响代码输出，只影响类型检查

[`Compiler Options`]: https://www.typescriptlang.org/docs/handbook/compiler-options.html
