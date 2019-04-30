---
title: will-change属性提高页面滚动、动画渲染性能(译)
date: 2019-04-30 09:48:09
tags: 
- css3
- 渲染性能
---

参考链接：

* [fix-scrolling-performance-css-will-change-property](https://www.fourkitchens.com/blog/article/fix-scrolling-performance-css-will-change-property/)
* [introduction-css-will-change-property](https://www.sitepoint.com/introduction-css-will-change-property/)
* [MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/will-change)

## 问题引入

最开始是为了解决一个滚动引起的性能问题：一个很大的元素，有一张视窗大小的背景图，使用`background-attachment: fixed`让它在滚动时一直处于可视区内，实现*视觉差*的效果。

### `fixed`带来的问题

使用`background-attachment: fixed`会导致每次用户滚动时都会引发绘制操作。为什么？页面需要重新定位内容，但它的背景图却应该仍在原来的位置不动，浏览器必须在相对于它实际DOM元素的新的位置上重绘。这是个很大的性能问题。

### 诊断问题

使用*Chrome DevTools*来查看*Frame*的绘制性能。

优化前：

![优化前](https://www.fourkitchens.com/wp-content/uploads/2017/01/fourkitchens-frontpage-before_0.png)

优化后：

![优化后](https://www.fourkitchens.com/wp-content/uploads/2017/01/fourkitchens-frontpage-after_0.png)

### 解决问题

使用`will-change`属性。

原始的CSS：

```scss
.what-we-do-cards {
  @include clearfix;
  border-top: 10px solid rgba(255, 255, 255, .46);
  background-color: white;
  background: url('/img/front/strategy.jpg') no-repeat center center;
  background-attachment: fixed;
  background-size: cover;
  color: $white;
  padding-bottom: 4em;
}
```

我们的背景图片使用了两个*GPU*敏感的CSS特性：`background-size: cover`和`background-attachment: fixed`。

GPU友好的CSS：

```scss
.what-we-do-cards {
  @include clearfix;
  border-top: 10px solid rgba(255, 255, 255, .46);
  color: $white;
  padding-bottom: 4em;
  overflow: hidden; // added for pseudo-element
  position: relative; // added for pseudo-element

  // Fixed-position background image
  &::before {
    content: ' ';
    position: fixed; // instead of background-attachment
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    background-color: white;
    background: url('/img/front/strategy.jpg') no-repeat center center;
    background-size: cover;
    will-change: transform; // creates a new paint layer
    z-index: -1;
  }
}
```

最重要的部分是伪元素上的`will-change: transform`属性。这个属性的作用是跟浏览器说：“hi浏览器，这个元素在未来会改变，请在新的层绘制它，这样它的邻居不会影响它的绘制。“

## 深入`will-change`

### 背景

很多前端工程师依赖CSS3 `transitions`，`transforms`和`animations`来增加新的层，现在我们只使用少量的CSS就可以实现顺畅漂亮的动画。如果你用过这些CSS属性，你很可能已经了解`CPU`，`GPU`和硬件加速了。下面我们快速过一下这些概念：

* The CPU, or the Central Processing Unit, is the piece of hardware that processes pretty much every computer operation. It’s otherwise known as the motherboard.
* The GPU, or the Graphics Processing Unit, is the piece of hardware associated with processing and rendering graphics. The GPU is designed to perform complex graphical computations and offloads some serious process weight from the CPU.
* Hardware acceleration is a general term for offloading CPU processes onto another dedicated piece of hardware. In the world of CSS transitions, transforms, and animations, it implies that we’re offloading the process onto the GPU, and hence speeding it up. This occurs by pushing the element to a layer of its own, where it can be rendered independently while undergoing its animation.

它是怎么提高性能和动画质量的呢？在基于`webkit`的浏览器中，当执行一下CSS操作时，经常会看到闪烁，即2D `transform`和`animation`。我们需要欺骗浏览器，让它把这些操作当作3D来执行，这样就可以把这些操作从CPU转移到GPU上做计算。因为3D的计算会自动放到GPU上。

### 什么是`will-change`？

CSS 属性 `will-change` 为web开发者提供了一种告知浏览器该元素会有哪些变化的方法，这样浏览器可以在元素属性真正发生变化之前提前做好对应的优化准备工作。 这种优化可以将一部分复杂的计算工作提前准备好，使页面的反应更为快速灵敏。

用好这个属性并不是很容易：

* **不要将 `will-change` 应用到太多元素上：**浏览器已经尽力尝试去优化一切可以优化的东西了。有一些更强力的优化，如果与 `will-change` 结合在一起的话，有可能会消耗很多机器资源，如果过度使用的话，可能导致页面响应缓慢或者消耗非常多的资源。

* **有节制地使用：**通常，当元素恢复到初始状态时，浏览器会丢弃掉之前做的优化工作。但是如果直接在样式表中显式声明了 `will-change` 属性，则表示目标元素可能会经常变化，浏览器会将优化工作保存得比之前更久。所以最佳实践是当元素变化之前和之后通过脚本来切换 `will-change` 的值。

* **不要过早应用 `will-change` 优化：**如果你的页面在性能方面没什么问题，则不要添加 `will-change` 属性来榨取一丁点的速度。 `will-change` 的设计初衷是作为最后的优化手段，用来尝试解决现有的性能问题。它不应该被用来预防性能问题。过度使用 `will-change` 会导致大量的内存占用，并会导致更复杂的渲染过程，因为浏览器会试图准备可能存在的变化过程。这会导致更严重的性能问题。

* **给它足够的工作时间：**这个属性是用来让页面开发者告知浏览器哪些属性可能会变化的。然后浏览器可以选择在变化发生前提前去做一些优化工作。所以给浏览器一点时间去真正做这些优化工作是非常重要的。使用时需要尝试去找到一些方法提前一定时间获知元素可能发生的变化，然后为它加上 `will-change` 属性。

| 定义 | value |
| --- | --- |
| 初始值 | auto |
| 适用元素| all elements |
| 是否是继承属性 | 否 |
| 适用媒体 | all |
| 计算值 | as specified |
| Animation type | discrete |
| 正规顺序 | the unique non-ambiguous order defined by the formal grammar |

#### 语法

> 标准语法: `auto` | `<animateable-feature>`
> 其中
> `<animateable-feature> = scroll-position | contents | <custom-ident>`
>
> ```html
will-change: auto
will-change: scroll-position
will-change: contents
will-change: transform        // Example of <custom-ident
will-change: opacity          // Example of <custom-ident>
will-change: left, top        // Example of two <animateable-feature>

will-change: unset
will-change: initial
will-change: inherit
```

#### 取值

`auto`
表示没有特别指定哪些属性会变化，浏览器需要自己去猜，然后使用浏览器经常使用的一些常规方法优化。

`<animateable-feature>` 可以是以下值：

`scroll-position`
表示开发者希望在不久后改变滚动条的位置或者使之产生动画。

`contents`
表示开发者希望在不久后改变元素内容中的某些东西，或者使它们产生动画。

`<custom-ident>`
表示开发者希望在不久后改变指定的属性名或者使之产生动画。如果属性名是简写，则代表所有与之对应的简写或者全写的属性。

#### 示例

```css
.sidebar {
  will-change: transform;
}
```

以上示例在样式表中直接添加了 `will-change` 属性，会导致浏览器将对应的优化工作一直保存在内存中，这其实是不必要的，前面我们已经看过为什么应该避免这样的做法。下面是另一个展示如何使用脚本正确地应用 `will-change` 属性的示例，在大部分的场景中，你都应该这样做。

```javascript
var el = document.getElementById('element');

// 当鼠标移动到该元素上时给该元素设置 `will-change` 属性
el.addEventListener('mouseenter', hintBrowser);
// 当 CSS 动画结束后清除 `will-change` 属性
el.addEventListener('animationEnd', removeHint);

function hintBrowser() {
  // 填写上那些你知道的，会在 CSS 动画中发生改变的 CSS 属性名们
  this.style.willChange = 'transform, opacity';
}

function removeHint() {
  this.style.willChange = 'auto';
}
```

但是，如果某个应用在按下键盘的时候会翻页，比如相册或者幻灯片一类的，它的页面很大很复杂，此时在样式表中写上 `will-change` 是合适的。这会使浏览器提前准备好过渡动画，当键盘按下的时候就能立即看到灵活轻快的动画。

```css
.slide {
  will-change: transform;
}
```
