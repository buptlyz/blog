---
title: 第一篇 Hexo使用说明
---
Hexo的使用说明，包括一些常用命令。

## 开发流程

### 初始化项目
全局安装hexo-cli：`npm install -g hexo-cli`
生成项目：`hexo init <folder name>`
进入文件目录并初始化：`cd <folder name> && yarn/npm i`

### 维护
拉仓库代码 `git clone git@github.com:buptlyz/blog.git`
进入文件目录并初始化 `cd blog && yarn` 或者 `cd blog && npm install`

## Hexo常用命令

### 创建新的post

``` bash
$ hexo new "我的新post"
```

更多信息: [写作](https://hexo.io/docs/writing.html)

### 启动服务

``` bash
$ hexo server
```

更多信息: [服务](https://hexo.io/docs/server.html)

### 生成静态文件

``` bash
$ hexo generate
```

更多信息: [生成](https://hexo.io/docs/generating.html)

### 部署到远程站点

``` bash
$ hexo deploy
```

更多信息: [部署](https://hexo.io/docs/deployment.html)
