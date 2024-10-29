---
title: CSS介绍
date: 2018-01-13 15:32:04
tags: [CSS,Notes]
---
## 历史与概述

> CSS（Cascading Style Sheets，层叠样式表）， 是一种[样式表](https://developer.mozilla.org/zh-CN/docs/Web/API/StyleSheet)语言，用来描述 HTML 或 XML（包括如 SVG、XHTML 之类的 XML 分支语言）文档的呈现。CSS 描述了在屏幕、纸质、音频等其它媒体上的元素应该如何被渲染的问题。

- 1994年哈肯·利缇提出了 CSS 的最初建议，而当时伯特·波斯（Bert Bos）正在设计一个名为Argo的浏览器，于是他们决定一起设计 CSS 。
- 1995年他们在 WWW 网络会议上 CSS 又一次被提出，波斯演示了 Argo 浏览器支持 CSS 的例子，哈肯也展示了支持 CSS 的 Arena 浏览器。
- 1996年底，CSS 初稿已经完成，同年12月，层叠样式表的第一份正式标准（Cascading style Sheets Level 1）完成，成为 W3C 的推荐标准。
- 1997年初，W3C 开始接管 CSS ，并组织了专门管 CSS 的工作组，其负责人是克里斯·里雷。他们开始讨论第一版中没有涉及到的问题。
- 1998年5月 W3C 发表了 CSS 2 ，并很快又发表了 CSS 2.1 修改了 CSS 2 的一些错误，消除了其中基本不支持的内容和增加了一些已有的浏览器的扩展内容，此时 CSS 开始流行。
- 从2011年开始，CSS 被分为多个模块单独升级，统称为 CSS 3。

<br>

## 引入方法
我们想要让文字居中且变红，则可以用以下四种方法实现：
### 内联样式
```
<!-- 在 CSS 还没发表之前，添加样式的方法 -->
<p><center><font color="red">你真棒！</font></center><p>
<!-- CSS 发表之后的方法 -->
<p style="text-align:center;color:red">你真棒！</p>
```
<small>注意：`<center>` 和 `<font>` 标签已过时，不建议使用。</small>
### `<style>` 标签
```
<head>
    <style>
        p{
            color: red;
            text-align: center;
        }
    </style>
</head>
<body>
    <p>你真棒！</p>
</body>
```
### `<link>` 标签引入外部文件

<small>假设本文件为 index.html ，它所在的文件夹下有一个用来写样式的 style.css 文件。</small>
```
<!-- 这是 index.html 的代码 -->
<head>
    <link rel="stylesheet" href="./style.css">
</head>
<body>
    <p>你真棒！</p>
</body>
```
```
/* 这是 style.css 的代码 */
p{
    text-align: center;
    color: red;
}
```
- 此外，还可以在 A.css 文件中使用 **`@import url(./B.css)`** 来引入另一个 CSS 文件，例如 B.css ，这样使得 HTML 文件渲染 A.css 前会先渲染 B.css 。

<small>假设 index.html 和 style.css 所在文件夹下还有个 anotherstyle.css 文件。现需在 style.css 中引入 anotherstyle.css ，使得 index.html 页面的背景色变为`#666`。</small>
```
/* 这是style.css的代码 */
@import url(./anotherstyle.css)
p{
    text-align: center;
    color: red;
}
```
```
/* 这是anotherstyle.css的代码 */
body{
    backgroud-color: #666;
}
```
<br>

## 元素的高度
- 内联元素的高度是由字体及其参数决定，无法设置宽高。`line-height` 的属性值可以决定内联元素的高度，例如 `line-height:20px` 那么 `<span>` 的高度也为 `20px` 。
- 块级元素的高度是由其文档流元素高度的总和决定。

> 文档流指文档内元素的流动方向。
> **块级元素从上往下流动**。
> **内联元素从左往右流动**，如果流动遇到阻碍（容器宽度不够）则换行继续从左往右流动。但如果是一个英文单词，则无论宽度是否已经超出容器宽度，都不将分割成两行，除非添加 [`word-break: break all`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/word-break) 属性。

<br>

## 如何学习CSS
- [MDN](https://developer.mozilla.org/zh-CN/search?q=CSS&topic=apps&topic=html&topic=css&topic=js&topic=api&topic=canvas&topic=svg&topic=webgl&topic=mobile&topic=webdev&topic=http&topic=webext) + keyword
- [阮一峰网络日志](http://www.ruanyifeng.com/blog/search.html?cx=016304377626642577906%3Ab_e9skaywzq&cof=FORID%3A11&ie=UTF-8&q=css&sa.x=0&sa.y=0) + keyword
- [张鑫旭的博客](http://www.zhangxinxu.com/wordpress/category/css/)
- [CSS-TRICKS](https://css-tricks.com/) + keyword
- [Codrops](https://tympanus.net/codrops/css_reference/)
- [CSS 2.1 中文规范文档](http://cndevdocs.com/)
- 《CSS揭秘》（Lea Verou著）

<br><hr><br>

## 一些小重点（未完待续...）
1. 页面默认字体大小为**16px**
2. 用 `float` 做横向结构需要给所有子元素添加 `float:left` ，给其父元素添加 `.clearfix::after{ content:""; display:block; clear:both; }` 。
