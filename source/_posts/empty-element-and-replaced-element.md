---
title: 空元素是什么？可替换元素又是个啥？
date: 2018-01-12 11:32:04
tags: [Notes]
---
## 空元素
> An empty element is an element from HTML, SVG, or MathML that cannot have any child nodes (i.e., nested elements or text nodes).

这是来自 [MDN](https://developer.mozilla.org/en-US/docs/Glossary/Empty_element) 的解释。按照我的理解，用大白话说出来就是，在 [HTML](https://developer.mozilla.org/zh-CN/docs/Web/HTML)、[SVG](https://developer.mozilla.org/zh-CN/docs/Web/SVG) 或 [MathML](https://developer.mozilla.org/zh-CN/docs/Web/MathML) 中，空元素指这个元素节点里不存在子节点，这个子节点指 `元素节点`（` element node `）或者 `文本节点`（` text node `）。

<small>PS：关于 DOM 节点的概念，如果你不了解，可以戳 [阮一峰的《JavaScript 标准参考教程(alpha)》中 6.1 概述](http://javascript.ruanyifeng.com/dom/node.html#toc2) 来学习。</small>

举例说明，
在 HTML 中有以下空元素：
- [`<area>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/area)
- [`<base>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/base)
- [`<br>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/br)
- [`<col>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/col)
- [`<colgroup>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/colgroup) when the [`span`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/colgroup#attr-span) is present
- [`<command>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/command) <strong style="color:red">Obsolete</strong>
- [`<embed>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/embed)
- [`<hr>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/hr)
- [`<img>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/img)
- [`<input>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input)
- [`<keygen>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/keygen)  <strong style="color:red">Obsolete</strong>
- [`<link>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/link)
- [`<meta>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/meta)
- [`<param>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/param)
- [`<source>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/source)
- [`<track>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/track)
- [`<wbr>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/wbr)

此外，在这些空元素上使用一个闭标签是无效的。如 `<input type="text"></input>` 是错误的。

在 SVG 中有 [`<font-face-name>`](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/font-face-name) 、[`<hkern>`](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/hkern) 、[`<vkern>`](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/vkern) 等。

在 MathML 中有 [`<mspace>`](https://developer.mozilla.org/zh-CN/docs/Web/MathML/Element/mspace) 、[`<semantics>`](https://developer.mozilla.org/zh-CN/docs/Web/MathML/Element/semantics) 等。


## 可替换元素
{% blockquote MDN https://developer.mozilla.org/en-US/docs/Web/CSS/Replaced_element Replaced element %}
In CSS, a replaced element is an element whose representation is outside the scope of CSS. In other words, these are external objects whose representation is independent of the CSS formatting model.
{% endblockquote %}

<br>看懂了吗？看懂了吗？不懂？那我给你看下MDN的[中文解释](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Replaced_element)。

{% blockquote %}
CSS 里，可替换元素（replaced element）的展现不是由CSS来控制的。这些元素是一类外观渲染独立于CSS的外部对象。
{% endblockquote %}

现在懂了？还是不懂？

Emmm...

[Maybe it would help.](https://www.cnblogs.com/WebShare-hilda/p/4713890.html)

 `替换元素` 是浏览器根据其标签的元素与属性来判断显示具体的内容的，CSS并不需要考虑渲染替换元素，它的展现独立于CSS，这就是“independent of the CSS formatting model”的意思。此外，不像 `非替换元素` 一样，它们将内容直接告诉浏览器，而检查替换元素的代码时可以发现它并没有实际的内容。

这是一个来自 W3C 关于 [Replaced element](https://www.w3.org/TR/CSS21/conform.html#replaced-element) 的解释，如果你看得懂英文的话，结合W3C标准的原始定义会理解得更准确一点。

另外，有人提出，把 `Replaced element` 翻译成可替换元素是不准确的，就英文字面意思来说， replaced 是已被替换了的， `Replaced element` 不由CSS负责展示渲染，不是可以被替换的概念。原文请戳[这里](http://openwares.net/2014/03/14/css_replaced_element/)。