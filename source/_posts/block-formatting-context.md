---
title: BFC（块格式化上下文）的理解及其应用
date: 2018-10-15 20:14:31
tags: [Notes, CSS]
---

## 理解 BFC
BFC（Block Formatting Context, 块格式化上下文），根据 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context) 的解释，它是布局过程中生成块级盒子的区域，也是浮动元素与其他元素的交互限定区域。看这个定义是真的很让人摸不着头脑，简单来说， BFC 就是页面中的一个独立容器，一个页面可以有很多的这样的容器，容器里的子元素与外界互不干扰。先看以下例子，你或许就能知道 BFC 到底是何方神圣了：

- 例一、现有一个 `div.parent` 包裹着它的儿子 `div.child` ：
    ```html
    <div class="parent">
      <div class="child"></div>
    </div>
    ```
    ```css
    .parent {
      outline: 1px solid red;
    }
    .child {
      width: 150px;
      height: 150px;
      background-color: #ccc;
    }
    ```
    此时 `div.parent` 的高度即为其儿子 `div.child` 的高度 `150px` 。如果为 `div.child` 添加左浮动，那么 `div.parent` 就无法包住它的儿子 `div.child` ，其高度变为 `0` （如下图所示）。
    ![div.child左浮动](https://i.loli.net/2018/10/15/5bc4589889eaa.png)

    但是，如果为 `div.parent` 创建一个 BFC ，就可以使它包住其儿子。如：
    ```css
    .parent {
      outline: 1px solid red;
      display: flow-root;
    }
    .child {
      float: left;
      width: 150px;
      height: 150px;
      background-color: #ccc;
    }
    ```
    ![例一](https://i.loli.net/2018/10/15/5bc458988a369.png)

- 例二、现有两个兄弟 `div.left` 和 `div.right` ，我为 `div.left` 添加左浮动，使得他们能够并排排列：
    ```html
    <div class="left">div.left</div>
    <div class="right">div.right</div>
    ```
    ```css
    .right {
      height: 150px;
      outline: 1px solid red;
    }
    ```
    结果发现，即使 `div.left` 设置了右边距，这两兄弟还是重叠在了一起，如下图所示：
    ![div.left与div.right重叠](https://i.loli.net/2018/10/15/5bc45cc291c2f.png)

    若想将两者分离开来，我们可以为 `div.right` 创建一个 BFC ，即：
    ```css
    .right {
      height: 150px;
      outline: 1px solid red;
      display: flow-root;
    }
    ```
    ![div.right的BFC作用](https://i.loli.net/2018/10/15/5bc45d9977a18.png)

通过以上两个例子，我们初步了解了 BFC 可以解决浮动元素对布局的影响，如浮动元素的父元素无法包住自身的问题、浮动元素与其他兄弟元素重叠的问题。此外，它还可以阻止 margin collapse（坍塌？不知道该如何翻译..看例子吧）的问题。例如：
```html
<div class="outer">
  <p>first paragraph</p>
  <p>second paragraphs</p>      
</div>
```
```css
div.outer {
  outline: 1px solid red;
}
p {
  outline: 2px solid #000;
  margin: 20px 0 20px 0;
}
```
我们可以看到，在 `p` 元素和其父元素 `div.outer` 的边缘之间没有任何东西的情况下， `div.outer` 像是塌了一样顶部直接了第一个 `p` 元素的顶部齐平，同理底部。
![margin collapse](https://i.loli.net/2018/10/15/5bc46f41cdb3b.png)

如果我们为 `div.outer` 创建 BFC ，那么就可以解决这个问题：
```css
div.outer {
  outline: 1px solid red;
  display: flow-root;
}
```
![创建BFC解决margin collapse问题](https://i.loli.net/2018/10/15/5bc47139ebc22.png)

细心的同学可以发现，尽管 `div.outer` 被撑起来了，但相邻的两个 `p` 元素还是发生了 margin 的重叠。这就是 BFC 的布局规则之一：同一 BFC 内的两个相邻块元素上下 margin 会重合。若想解决这个问题，只需在其中一个 `p` 元素外创建一个 BFC 包裹器即可：
```html
<div class="outer">
  <p>first paragraph</p>
  <div class="wrapper">
    <p>second paragraphs</p>
  </div>
</div>
```
```css
div.outer {
  outline: 1px solid red;
  display: flow-root;
}
p {
  outline: 2px solid #000;
  margin: 20px 0 20px 0;
}
div.wrapper {
  display: flow-root;
}
```
![BFC中的BFC](https://i.loli.net/2018/10/15/5bc47493e810f.png)


## 创建 BFC 的方法
在上面的所有例子中，我都使用 **`display: flow-root;`** 来创建 BFC ，当然还有其他的方法，例如：
- 根元素
- `float` 属性不为 `none`
- `overflow` 属性不为 `visible` 的块元素
- `position: absolute;` 或 `position: fixed;`
- `display: flex;` 或 `display: inline-flex;`
- `display: grid;` 或 `display: inline-grid;`
- `display: inline-block;`, `display: table-cell;` 或 `display: table-caption;`
- `column-span: all;`
- ... [view more on MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)

这些大法虽好，但很容易产生副作用。例如，如果我们使用 `overflow: auto;` 来创建一个 BFC ，在某些情况下，你可能会发现有一个多余显示的滚动条或者像我们上述例子中的 `outline` 会被剪切掉。而 [***flow-root***](https://drafts.csswg.org/css-display/#valdef-display-flow-root) 是 CSS2 中专门设计为创建 BFC 的属性，它不会产生多余的副作用，所以建议使用这个属性来代替其他方法。



## 不使用 BFC 清除浮动（题外扩展）
若要清除浮动，我们可以使用 [clearfix hack](https://css-tricks.com/snippets/css/clear-fix/) ，只需将 `clearfix` 加到浮动元素的父元素的 `class` 属性中即可，而不一定非得创建 BFC 来达到清除浮动的效果。例如例一可以通过以下方式达到同样的效果：
```html
<div class="parent clearfix">
  <div class="child"></div>
</div>
```
```css
.parent {
  outline: 1px solid red;
}
.child {
  float: left;
  width: 150px;
  height: 150px;
  background-color: #ccc;
}
.clearfix::after {
  content: '';
  display: block;
  clear: both;
}
```


-------

本文完。

若文中有错误还请指正与包涵！

原文链接：https://caijialinxx.github.io/2018/10/15/block-formatting-context/

转载请注明出处。

参考资料：
- [Understanding CSS Layout And The Block Formatting Context](https://www.smashingmagazine.com/2017/12/understanding-css-layout-block-formatting-context/)
- [块格式化上下文 - Web开发者指南 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)
- [前端精选文摘：BFC 神奇背后的原理](https://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html)