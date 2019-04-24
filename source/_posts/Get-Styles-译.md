---
title: Get Styles(译)
date: 2019-04-22 22:41:23
tags:
---

[原文链接](https://www.quirksmode.org/dom/getstyles.html)

有时你希望查看默认文档的样式。比如，你给一个paragraph定义50%的宽度，但怎样才能查看它在浏览器里的像素值呢？

此外，有时你想查看一个被嵌入样式/外链样式定义的元素的样式。而`style`属性只返回元素的内联样式，所以你必须想其它办法来获取样式。

## offset

Before going to the tricky bits, first a nice shortcut that has been inserted into both Mozilla and Explorer: offsetSomething. Using these few properties you can read out the most important information about the styles the paragraph currently has.

As an example, get the offsetWidth of the test paragraph. You'll see how many pixels its width is at the moment. To test it some more, resize the window (the paragraph, having a width of 50%, will also resize) and try again.

The script is quite simple:
