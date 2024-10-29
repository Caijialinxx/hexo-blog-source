---
title: 面试常考题之 HTML
date: 2018-07-17 16:10:45
tags: [面试, HTML]
---

## HTML 语义化的理解
> A semantic element clearly describes its meaning to both the browser and the developer.
一个语义化的元素能够清晰地向浏览者和开发者描述它的意义。

常见的语义化标签有： `<p>` 、 `<h1>` 、 `<button>` 、 `<article>` 、 `<aside>` 、 `<header>` 、 `<footer>` 、 `<main>` 、 `<nav>` 、 `<section>` 等。

- 「语义化」概念的出现
  - 最开始的 HTML 页面是由 PHP 后端写的，他们不会 CSS ，于是只能使用 table 来布局。然而 table 是用来展示表格的，这种投机取巧的使用方法严重违反了 HTML 语义化原则。
  - 后来，有了会写 CSS 的前端之后，他们就使用 DIV + CSS 来实现页面布局，主要是用 `float` 和 `position` 属性。看起来好了一点，但整个页面的结构都是 DIV ，也非常的不可观。
  - 随着发展，前端渐渐变得专业，为了改善这种杂乱的结构，「语义化」这一概念被提出，前端尽量使用 `<p>` 、 `<h1>` 这种有意义的标签来展示内容。此外， HTML5 为我们提供了更多的语义化标签，例如 `<header>` 、 `<footer>` 、 `<main>` 、 `<nav>` 、 `<section>` 等。

### 总的来说
  HTML 语义化就是使用有意义的标签来展示内容，例如 `<p>` 表示段落， `<aside>` 表示侧边栏， `<nav>` 表示导航。这使得页面结构易于阅读、方便理解，此外，还能有益于 [SEO](https://baike.baidu.com/item/%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E%E4%BC%98%E5%8C%96/3132?fromtitle=seo&fromid=102990&fr=aladdin) （Search Engine Optimization，搜索引擎优化）、其他设备的解析（如屏幕阅读器、盲人阅读器、移动设备）。



## meta viewport 的原理
```html
<meta name='viewport' content='width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0'>
```
最初，网页只会在 PC 端展现，所以页面宽度是适应大多数 PC 端的。后来，随着智能手机（iPhone 3GS）的出现，原本为 PC 端设计的网页在小屏的手机上就“胖到挤不下”，用户只能不断移动这些网页来达到浏览的目的。后来乔布斯的工程师想了一个办法：他们让手机宽度模拟成 980px ，这样页面就能缩小了，用户想要浏览某个部分只需要手动放大。
但是随着智能手机的普及，这种不方便用户的功能显然不够贴心。 `<meta name='viewport'>` 的出现就是为了解决这个问题：

| 属性名 | 属性值 | 描述 |
| - | - | - |
| width | [number] / device-width | 控制 viewport 的大小，可以指定的一个数字，如 600，或者特殊的值，如 device-width 为设备的宽度（单位为缩放为 100% 时的 CSS 的像素）。 |
| height | [number] / device-height | 和 width 相对应，指定高度。 |
| initial-scale | [number] | 初始缩放比例，也即是当页面第一次加载时的缩放比例。 |
| maximum-scale | [number] | 允许用户缩放到的最大比例。 |
| minimum-scale | [number] | 允许用户缩放到的最小比例。 |
| user-scalable | yes / no | 用户是否可以手动缩放。 |

### 总的来说
我们可以通过设置 `<meta name='viewport' content='width=device-width, initial-scale=1.0'>` 来使得页面自适应屏幕的宽度，并且不被缩小。


## canvas 的使用
这里有一个我自己做的项目示例 [Canvas 画板](https://caijialinxx.github.io/jirengu-homework/canvas/)，去玩一下呗~

这里总结一下 `<canvas>` 元素中常用的 API 及用法。

首先我们要在 HTML 中添加一个 canvas 元素：
```html
<canvas id="canvas"></canvas>
```
然后在 JavaScript 中取到这个元素，就可以执行一些操作：
```js
// 获取 canvas 元素
var canvas = document.getElementById('canvas')
// 「常用 API」 1. getContext() 访问绘画上下文
var ctx = canvas.getContext('2d')

// 「常用 API」 2. fillStyle 设置图形的填充颜色
ctx.fillStyle = 'orange'
// 「常用 API」 3. strokeStyle 设置图形的轮廓颜色
ctx.strokeStyle = 'red'
// 「常用 API」 4. strokeStyle 设置图形的轮廓颜色
ctx.lineWidth = 2

/* 绘制矩形 */
// 「常用 API」 5. fillRect(x, y, width, height) 绘制一个填充的矩形
ctx.fillRect(25,25,100,100)
// 「常用 API」 6. clearRect(x, y, width, height) 清除指定矩形区域，让清除部分完全透明
ctx.clearRect(45,45,60,60)
// 「常用 API」 7. strokeRect(x, y, width, height) 绘制一个矩形的边框
ctx.strokeRect(50,50,50,50)

/* 绘制路径 */
// 「常用 API」 8. beginPath() 新建一条路径
ctx.beginPath()
// 「常用 API」 9. moveTo(x, y) 将笔触移动到指定的坐标x以及y上
ctx.moveTo(160,75)
// 「常用 API」 10. lineTo(x, y) 绘制一条从当前位置到指定x以及y位置的直线
ctx.lineTo(220,125)
ctx.lineTo(220,25)
// 「常用 API」 11. closePath() 闭合路径，不是必须的
ctx.closePath()
// 「常用 API」 12. stroke() 通过线条来绘制图形轮廓。如果不调用 closePath() 则 stroke() 不会自动闭合
ctx.stroke()
// 「常用 API」 13. fill() 填充路径的内容区域生成实心的图形。即使不调用 closePath() 所有没有闭合的形状仍会自动闭合填充
ctx.fill()

/* 绘制圆形 */
ctx.beginPath()
/*
 * 「常用 API」 14. arc(x, y, radius, startAngle, endAngle, anticlockwise)
 *                  画一个以（x,y）为圆心， radius 为半径的圆，
 *                  从 startAngle 开始到 endAngle 结束，
 *                  按照 anticlockwise 给定的方向（默认为顺时针）来生成
 */
ctx.arc(300, 75, 50, 6, 2*Math.PI, true)
ctx.fill()
ctx.stroke()
```
你可以查看这个小 Demo 的[演示效果](http://jsbin.com/vekayejuyi/edit?js,output) ，并且可以尝试改动一下。

- 总结：
  - **`getContext()`**
  - `fillStyle`
  - `strokeStyle`
  - `beginPath()`
  - `closePath()`
  - `moveTo()`
  - `lineTo()`
  - `stroke()`
  - `fill()`
  ...

你还可以学习一下 MDN 中的 [canvas 教程](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial) ，里面还有[绘制文本](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Drawing_text)、[变形](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Transformations)等很有意思的教程。


## 细节常识题
1. 浏览器默认的字体大小
    - **16px**
2. `<strong>` 和 `<b>` 、 `<em>` 和 `<i>` 的区别
    - `<strong>` 和 `<b>` 都可以使字体变粗，但前者表示强调，它可以标明重点内容，而后者没有什么实际含义。
    - `<em>` 和 `<i>` 都可以使字体变斜体，但前者表示也是用来强调文本，经常使用在表示重要术语或引用等，而后者没有什么实际含义。
      ```html
      我是这个世界上<strong>最美的</strong>人。
      我想让这个字变<b>粗</b>。
      我喜欢 <em>Professional Javascript for Web Developer 3rd</em> 这本书。
      我想让这个字变<i>歪</i>。
      ```
      我是这个世界上<strong>最美的</strong>人。
      我想让这个字变<b>粗</b>。
      我喜欢 <em>Professional Javascript for Web Developer 3rd</em> 这本书。
      我想让这个字变<i>歪</i>。

    - 我们应尽量遵循 HTML 语义化，表示重要的内容时使用 `<strong>` 或 `<em>` ，避免单纯为了让字体变粗而使用 `<b>` ，或者单纯为了让字体变斜而使用 `<i>` 的情况，这些样式应该由 CSS 去控制。
    - 这里有一个来自 [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/strong#Usage_notes) 的比较，可以参考一下。
3. `href` 和 `src` 的区别
    - `href` 是 Hypertext Reference 的缩写，表示超文本引用，用来建立文档和外部资源之间的联系。常用的有 `<a>` 和 `<link>` 标签。
    - `src` 是 source 的缩写，是页面种必不可少的一部分，其指向的内容会嵌入到文档中当前元素所在的位置。浏览器在下载、解析这个元素的时候会暂停页面的加载和处理。常用的有 `<script>` 、 `<img>` 和 `<iframe>` 标签。
    - 总的来说就是 `src` 用于替换当前元素，而 `href` 用于建立文档与引用资源的联系。
4. `readonly` 和 `disabled` 的区别
    - `readonly` 表示这个控件是**只读**的，无法在页面上修改，但可以通过 JavaScript 修改。在提交表单时此控件的值会被传出去。
    - `disabled` 表示这个控件是**被禁用**的，同样无法在页面上修改内容（可以通过 JS 修改），在提交表单时该控件的内容不会被提交。
5. 块级元素与行内元素的区别
    - 默认情况下，块级元素会另起一行，而行内元素不会以新行开始。
    - 块级元素可以设置宽高，而行内元素的宽高由内容物决定。
    - 块级元素里面可以有行内元素或其他块级元素，而行内元素只能包含数据或其他行内元素。
    - 块级元素有 `<div>` 、 `<p>` 、 `<ul>` 、 `<header>` 、 `<form>` 等，而行内元素有 `<span>` 、 `<select>` 、 `<strong>` 、 `<input>` 、 `<button>` 、 `<textarea>` 等。
6. doctype 的作用
    - 声明文档类型，告诉浏览器要用什么文档类型规范来解析这个文档。