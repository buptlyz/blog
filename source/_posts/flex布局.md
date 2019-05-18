---
title: flex布局
date: 2019-05-18 16:16:57
tags:
- flex
---

## 概念

* 主轴
* 垂直轴

* flex container
* flex item

## 默认行为

`flex item`的默认行为：

* 按行排列 flex-direction: row;
* 沿着主轴起始边界开始排列 justify-content: flex-start;
* 主轴方向不会膨胀，但会缩小 flex-grow: 0; flex-shrink: 1; flex-basis: auto;
* 垂直轴会膨胀填满空间 align-items: stretch;
* flex-basis: auto;
* flex-wrap: nowrap;

## 属性默认值，可选值

### flex container

* flex-direction: row;
* flex-wrap: nowrap; (wrap)
* align-items: stretch; (flex-start, flex-end, center)
* justify-content: flex-start; (flex-end, center, space-around, space-between, space-evenly)

### flex item

* flex-basis: auto; (inherit, initial, unset)
* flex: initial; (= 0 1 auto) (auto = 1 1 auto, none = 0 0 auto)
