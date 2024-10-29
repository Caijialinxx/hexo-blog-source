---
title: 移动端的适配
date: 2018-05-25 19:40:00
tags: [面试, CSS]
---

腾讯曾经出过一个面试题：移动端该如何做适配？有位同学分享了他那次的面试经历，有兴趣可以[了解一下](https://github.com/Bless-L/MyBlog/blob/master/post/2016%E8%85%BE%E8%AE%AF%E5%AE%9E%E4%B9%A0%E7%94%9F%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E7%BB%8F%E5%8E%86%E5%8F%8A%E6%80%BB%E7%BB%93%EF%BC%88%E4%BA%8C%EF%BC%89.md)。

移动端在用户群中占了很大的比重，所以做好移动端的适配，基本是前端们必备的技能之一。移动端和PC端的最重要的区别就是
1. 视口大小

    移动端的视口宽度基本在 1000px 以内，手机端在 320px 至 425px 之间，若将PC端的页面在手机端中展示，最初的做法是用手机端的视口宽度去模拟PC端的 980px ，那么内容会被缩放，用户浏览时需要自行放大。
    
2. 交互方式
    - 移动端没有 hover 事件，PC端页面的 hover 在手机端中无法展现
    - 移动端有 touch 事件

那么，要想使得网页在移动端也能很好地展现出来，大概有以下几种方法：

## 移动端的适配总结
<small>把总结提到最前写是为了方便复习。</small>
1. <a href='#Viewport元标签'>viewport元标签</a>
2. <a href='#媒体查询（Media Queries）'>媒体查询</a>，自动探测屏幕宽度来加载相应的CSS
3. 尽量不使用绝对单位（如px），而使用相对单位（如<a href='#REM'>rem</a>、em）



## Viewport元标签
在移动端中常常会添加 meta viewport 来规定网页的显示形式。
> `<meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0">`

上面这行代码的意思是，网页宽度默认等于屏幕宽度（width=device-width），不允许用户缩放（user-scalable=no），网页初始大小占屏幕面积的100%（initial-scale=1.0）。
- `width` 设置视口宽度，device-width指设备宽度
- `user-scalable` 是否允许用户缩放， no 为不允许， yes 为允许
- `initial-scale` 设置页面地初始缩放比例
- `maximum-scale` 允许的最大缩放比例
- `minimum-scale` 允许的最小缩放比例



## 媒体查询（Media Queries）
- 样式表中的媒体查询。例如：
  ```css
  @media (max-width: 980px) {
    body {
      background-color: red;
    }
  }
  ```
  这段 CSS 代码规定了当媒体的宽度 ≤980px 时，背景色变成红色。

- `<link>` 标签中的媒体查询。例如：
  ```html
  <link rel='stylesheet' href='style.css' media='(max-width: 980px)'>
  ```
  这段代码表示当满足媒体查询的宽度 ≤980px 时， `style.css` 才会生效（页面加载时已下载好，等待条件满足时使用）。

  相较之下，第二个例子的媒体查询方法会更方便，即不同媒体查询结果下，引用不同的 CSS 文件，它能避免样式覆盖的问题。

- CSS 文件中的媒体查询。例如：
  ```css
  @import url("tinyScreen.css") (max-width: 400px);
  ```

[Click here to learn more about Media Queries](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Media_queries).



## 触摸屏设备的交互事件
- 没有 :hover 伪类

    因为在触摸屏设备中，用户无法像在 PC 端上一样，把鼠标移动到某个带有 :hover 伪类的元素上就能做出回应，所以我们需要为触摸屏设备以 click 事件来模拟 PC 端中的 hover 事件。例如：
    ```html
    <!-- PC 端 -->
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        #xxx {
          width: 50px;
          height: 50px;
          background-color: red;
        }
        #xxx:hover {
          background-color: blue;
        }
      </style>
    </head>
    <body>
      <div id="xxx"></div>
    </body>
    </html>
    ```
    这段代码实现的效果是：
    <pre style='white-space: normal'>
    <div id='xxx' style='width: 50px;height: 50px;background-color: red;'></div>
    <script type="text/javascript">
      xxx.onmouseenter = function(){
        xxx.style.backgroundColor = 'blue';
      };
      xxx.onmouseout = function(){
        xxx.style.backgroundColor = 'red';
      };
    </script>
    </pre>

    要想在触摸屏设备中实现类似效果，那么代码应该改写成：
    ```html
    <!-- 触摸屏设备 -->
    <!-- 相同部分已省略，script 加到 body 内最后 -->
    <script type="text/javascript">
      xxx.onclick = function(){
        if(xxx.style.backgroundColor === 'red'){
          xxx.style.backgroundColor = 'blue'
        } else {
          xxx.style.backgroundColor = 'red'
        }
      }
    </script>
    ```

- touch 事件

    在这里就不做深入研究，有兴趣可以看 MDN 的[触摸事件](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events)例子，以及[演示效果](https://developer.mozilla.org/samples/domref/touchevents.html)（请在触控设备中打开）。



## REM

> rem 是一个相对单位，这个单位代表**根元素 `<html>` 的 `font-size` 大小**。

例如定义了如下样式，那么 `<body>` 中的字体大小实际为 18px 。它不仅适用于字体大小，还可以用在 `width`、 `margin` 等的样式上。此外，它还可以与其他单位共存。
```html
<!DOCTYPE html>
<html>
<head>
  <style>
    html {
      font-size: 12px;
    }
    body {
      font-size: 1.5rem;
    }
    #div {
      width: 6rem;
      height: 6rem;
      border: 1px solid #999;
      padding: 5px;
    }
  </style>
</head>
<body>
  这是一段在 body 内的文字。
  <div id='div1'>这是一个在 body 内的 div 。</div>
</body>
</html>
```
代码结果：
<pre style='white-space: normal'>
<span style='font-size: 18px'>这是一段在 body 内的文字。</span>
<div style='width: 72px; height: 72px; border: 1px solid #999; padding: 5px;'>这是一个在 body 内的 div 。</div>
</pre>

> 插一个拓展知识点：有趣的是，若上面例子中， div1 的宽高的单位改为 `em` ，则它的实际宽高由 72px 变成 108px 。这是因为 `em` 表示元素的 `font-size` 计算值，div1 继承了父元素 `<body>` 的 `font-size` 值（18px），所以它的 1em 实际上等于 18px 。 [Click here to learn more about length units](https://developer.mozilla.org/en-US/docs/Web/CSS/length).


## Sass —— 强大的 CSS 扩展语言
使用 Sass 来开发 CSS 可以方便样式的编写。它不仅完全兼容 CSS ，还允许我们使用变量、嵌套等，甚至还可以声明函数来使用。

下面是一个 scss 文件，我们可以简单看看它的语法规则大概是怎么样的：
```scss
@function px2rem( $px ){  //自定义函数
  @return $px/$deviceWidth*10 + rem;
}
$deviceWidth: 640;        //变量以$符号开头

ul {
  width: px2rem(256);
  padding: px2rem(32);
  background-color: #eee;
  list-style: none;
  font-size: 1.2em;

  // 嵌套
  li {
    padding: 4px;
    border-bottom: 1px solid #ddd;
  }
  li:nth-child(3) {
    border-bottom: none;
  }
}
```
经过转换，上面的 scss 文件对应的 css 文件如下：
```css
ul {
  width: 4rem;
  padding: 0.5rem;
  background-color: #eee;
  list-style: none;
  font-size: 1.2em;
}
ul li {
  padding: 4px;
  border-bottom: 1px solid #ddd;
}
ul li:nth-child(3) {
  border-bottom: none;
}
```
[Click here to learn more about Sass](http://sass-lang.com/guide).
或者，需要中文的话，这里有一篇阮一峰老师学习[Sass的博客笔记](http://www.ruanyifeng.com/blog/2012/06/sass.html)，讲得很详细也一眼就能看懂。



结语：后期还会为本文章更新 Flex 布局。爸爸要去吃饭了。