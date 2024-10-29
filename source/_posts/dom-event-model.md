---
title: DOM 事件模型的发展
date: 2018-05-28 19:35:03
tags: [DOM, Notes, JavaScript]
---

## DOM Level 1 （背景）
W3C 为了统一各浏览器，梳理了微软和网景两大巨头的浏览器的事件监听模型，整合成第一个版本的标准（DOM Level 1）并发布。所以其实 DOM Level 1 只是标准出来之前已有的事件监听的集合。现在有以下代码，我们来尝试为这些按钮绑定事件，使得当点击这些按钮时，弹窗显示“hi”。
```html
<button id='A'>A</button>
<button id='B'>B</button>
<button id='C'>C</button>
<script>
  function sayHi(){ alert('hi') }
</script>
```

- 利用 HTML 事件属性来分配事件

  ```html
  <!-- A -->
  <button id='A' onclick='sayHi'>A</button>
  <!-- B -->
  <button id='B' onclick='sayHi()'>B</button>
  <!-- C -->
  <button id='C' onclick='sayHi.call()'>C</button>
  ```
  代码结果：
  <button id='A' onclick='sayHi'>A</button> <button id='B' onclick='sayHi()'>B</button> <button id='C' onclick='sayHi.call()'>C</button>
  <script>function sayHi(){ alert('hi') }</script>

  在上述代码中，只有 **B** 和 **C** 才能得到我们想要的结果。DOM Level 1 规定，当使用 HTML 事件属性绑定事件名时，必须带上英文括号。因为事件属性的值是指要运行的代码，一旦用户点击按钮，浏览器就 `eval('sayHi()')` 或者 `eval('sayHi.call()')` 。 `sayHi` 只是一个函数变量名，如果是按钮A，那么浏览器执行的是 `eval('sayHi')` ，这样只是运行了这个变量，并没有执行这个函数。所以切记正确写法为后两者。


- 利用 HTML DOM 来分配事件

  现在，让我们来看看在 js 中，我们又该如何利用 HTML DOM 来得到我们想要的结果。
  ```javascript
  function meetAgain(){
    alert('hi')
  }
  A.onclick = meetAgain           //A：正确
  B.onclick = meetAgain()         //B：错误
  C.onclick = meetAgain.call()    //C：错误
  ```
  上述代码中，只有 **A** 才是正确的。因为在 JavaScript 中， `onclick` 是一个属性， `meetAgain` 是一个函数对象， `A.onclick = meetAgain` 是将 `meetAgain` 函数的对象地址赋给 `onclick` 属性。而 `meetAgain()` 和 `meetAgain.call()` 表示执行函数，返回的结果是 `undefined` ，那么 B 和 C 的 `onclick` 的属性值为 `undefined` ，很显然这并不是我们要的结果。
  
所以 HTML 事件属性 和 HTML DOM 两者的区别：
> HTML 事件属性中分配事件时，需要在函数名后加上“()”，表示当用户点击时就执行这个函数；HTML DOM中为事件属性分配事件时，不需要在函数名后加“()”，表示将函数的对象地址赋给事件属性。



## DOM Level 2 （发展）
在 DOM Level 1 中我们可以看到，要为一个元素绑定事件，只能用 `onclick` 、`onmouseup` 等属性，这就带来了一个问题，一个属性只能绑定一个事件。DOM Level 2 中最重要的更新就是添加了 DOM 事件模型，使得一个属性可以绑定多个事件。此外，这是我们目前**最常用的 DOM 标准**。

- 重要的API：`target.addEventListener(type, listener, options)`

  具体看以下代码，来理解 `addEventListener()` 和 `onclick` 绑定事件的区别（ HTML 代码仍为本文开头的第一段代码）：
  ```javascript
  A.addEventListener('click', function (){
    console.log('A1')
  })
  A.addEventListener('click', function (){
    console.log('A2')
  })
  B.onclick = function (){
    console.log('B1')
  }
  B.onclick = function (){
    console.log('B2')
  }
  ```
  当用户分别点击 A按钮 和 B按钮 时，运行结果为：
  <pre>
  > "A1"
  > "A2"
  > "B2"
  </pre>

  我们发现，A 的 click 事件的两个结果都运行出来了，而 B 只运行了在后位绑定的事件函数。这是因为 `onclick` 属性是唯一的，若绑定了两个及以上的事件处理函数，则先绑定的会被新绑定的覆盖。而 `addEventListener()` 会让事件进入到事件监听队列（eventListeners）中，执行顺序依照队列先进先出的特性。所以不建议直接使用 `onclick` 属性来绑定事件处理函数，以免被无意覆盖。

- 与 `addEventListener()` 对应的是 `removeEventListener()` ，它可以将指定函数移出事件监听队列。
  ```javascript
  function func(){
    console.log('C')
  }
  C.addEventListener('click', func)
  C.removeEventListener('click', func)
  ```
  此时，点击 C按钮 时，并不会输出任何内容，因为 func 在进入事件监听事件队列后又马上被移出了，等不到用户点击。


> 事件流（Event flow）： IE 提出的事件冒泡（Event bubbling）、网景提出的事件捕获（Event capture）。

```html
<div id='head'>
  头
  <div id='face'>
    脸
    <div id='nose'>
      鼻子
    </div>
  </div>
</div>
```

- Q1：当点击“鼻子”时，头和脸有没有被点击到？

  ```javascript
  head.addEventListener('click', function (){
    console.log('头');
  })
  face.addEventListener('click', function (){
    console.log('脸');
  })
  nose.addEventListener('click', function (){
    console.log('鼻子');
  })
  ```
  根据上述代码，当点击“鼻子”时，控制台输出的结果是：
  <pre>
  > "鼻子"
  > "脸"
  > "头"
  </pre>

  由此可见，当点击“鼻子”时，“脸”和“头”也被点击到了。因为“鼻子”是“脸”和“头”的一部分，顺序是由里到外，由小到大。


- Q2：当点击“鼻子”时，三者的事件处理顺序是怎么样的？

  在上一个例子中，我们看到顺序是由里到外（由小到大）的，那么顺序只能是这一种吗？ W3C 给出了两种事件流机制，分别是冒泡流和捕获流。上一个例子就是冒泡流的例子。而要采用冒泡流还是捕获流，就由 `addEventListener()` 的第三个参数 options 决定 —— 当 options 为 `false` 时就是冒泡流（默认），为 `true` 时就是捕获流。

  ```javascript
  /* 冒泡阶段，忽略第三个参数则默认为 undefined ，undefined 为 false */
  head.addEventListener('click', function (){
    console.log('头：冒泡');
  })
  face.addEventListener('click', function (){
    console.log('脸：冒泡');
  })
  nose.addEventListener('click', function (){
    console.log('鼻子：冒泡');
  })

  /* 捕获阶段 */
  head.addEventListener('click', function (){
    console.log('头：捕获');
  }, true)
  face.addEventListener('click', function (){
    console.log('脸：捕获');
  }, true)
  nose.addEventListener('click', function (){
    console.log('鼻子：捕获');
  }, true)
  ```

  运行结果：
  <pre>
  > "头：捕获"
  > "脸：捕获"
  > "鼻子：冒泡"
  > "鼻子：捕获"
  > "脸：冒泡"
  > "头：冒泡"
  </pre>

  由这个运行结果，我们可以看出捕获阶段与冒泡阶段的执行顺序是相反的，它是由外到里，由大到小的，且“头”和“脸”都是先捕获后冒泡。而例外的是，“脸”却是先冒泡后捕获。

  其实，捕获阶段的事件是先于冒泡阶段处理的。而当事件流完成祖先元素及父元素的捕获阶段后，到达目标元素（target）时，事件处理顺序遵循函数绑定的顺序处理（即按顺序执行代码）而不是先捕获后冒泡，这个阶段被称为“目标阶段”。理解过程如下图：
  ![event-flow.png](https://i.loli.net/2018/05/16/5afbeb914da76.png)