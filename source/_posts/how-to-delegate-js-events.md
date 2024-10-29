---
title: 事件委托——让事件监听更便捷
date: 2020-4-24 15:30:45
tags: [Notes, JavaScript, DOM, 面试]
---

我们知道，在 HTML 中为一个元素绑定事件，需要对这个元素添加事件监听，如 `onclick` 属性或者 `addEventListener()` 。
```html
<ul id="list">
  <li id="item-1" onclick="console.log('I am Item 1)">Item 1</li>
  <li id="item-2" onclick="console.log('I am Item 2)">Item 2</li>
  <li id="item-3" onclick="console.log('I am Item 3)">Item 3</li>
  <li id="item-4" onclick="console.log('I am Item 4)">Item 4</li>
  <li id="item-5" onclick="console.log('I am Item 5)">Item 5</li>
  <li id="item-6" onclick="console.log('I am Item 6)">Item 6</li>
</ul>
```
```js
document.querySelector('#item-1').addEventListener('click', () => { console.log('I am Item 1') })
document.querySelector('#item-2').addEventListener('click', () => { console.log('I am Item 2') })
document.querySelector('#item-3').addEventListener('click', () => { console.log('I am Item 3') })
document.querySelector('#item-4').addEventListener('click', () => { console.log('I am Item 4') })
document.querySelector('#item-5').addEventListener('click', () => { console.log('I am Item 5') })
document.querySelector('#item-6').addEventListener('click', () => { console.log('I am Item 6') })
```

这样看起来实在是太笨啦！并且在动态增减列表元素的场景下，它无法及时地添加或移除事件监听。怎么让事情变得更简单些呢？那就是使用事件委托来进行监听：
```js
document.querySelector('#list').addEventListener('click', (e) => {
  if (e.target.tagName === 'LI') {
    console.log(`I am ${e.target.textContent}`)
  }
})
```

代码是不是立马就变得清爽起来～我们只用一个事件监听就实现在每个 `<li>` 元素点击时都能执行某些操作。

但现实中，我们的页面结构往往不会这么简单，需要产生交互的元素里面可能包含了很多子元素，例如图标、被其他标签包裹的文本等，这使得 `event.target` 不一定是目标元素。这个时候我们就得对当前点击的元素逐层向上寻找父元素，若找到目标元素，则说明我们需要执行对应的操作。所以，我们需要写一个更通用的事件委托：
```js
function delegate(eventType, targetSelector, fn) {
  document.addEventListener(eventType, (e) => {
    let currentEl = e.target;
    while(!currentEl.matches(targetSelector)) { // 通过选择器来匹配，而不是写死tagName匹配
      if(currentEl === document.documentElement) { // 当已经查到顶后，及时停止向上查找
        currentEl = null;
        break;
      }
      currentEl = currentEl.parentElement;      
    }
    if(currentEl) {
      fn(currentEl);
    }
  })
}

delegate('click', 'li', () => { ... })
```

最后，你可以到这里来 [Try And Code](https://jsbin.com/negebekuso/3/edit?html,js,output) 一下。



