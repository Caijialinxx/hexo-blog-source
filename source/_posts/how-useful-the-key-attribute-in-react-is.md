---
title: 原来React里的key这么有用！
date: 2019-9-26 15:56:40
tags: [React, Notes]
---


相信React用户在撸代码的日常中，一定遇到过这个报错：

![React中常见的key属性警告](https://i.loli.net/2019/10/16/CJWrpevz89xTK7Z.png)

嗯，没错了，熟悉的配方，熟悉的报错～ 有些小伙伴看到是「Warning」也就置之不理了，有些凭经验反手给元素加上一个 `key={唯一的值}` 属性就迅速解决这个报错了。

不知道有多少小伙伴知道为什么React这么看重这个 `key` 属性，反正我以前凭直觉添加个唯一的id，再不济用索引就完事了。今天才明白，原来 `key` 这么有用，而且用好了还能避免性能问题甚至是 Bug ！



## Key 为何而来？
在正式开始解释 `key` 属性之前，我得先说说React的Diffing算法。为了避免本文重心偏离及篇幅太长，我打算概括性地介绍一下这个过程：

首先检查根元素的类型。
1. 如果类型不一样，则卸载整个旧的DOM树，并重建新的DOM树。
2. 如果类型一样，React会检查两者的属性。如果属性有改变，就更新相应的属性。
3. 在比较完该节点后，递归地检查其子节点。默认情况下，React会同时遍历旧树和新树的子节点列表，在存在差异时则触发一个变化。
    例如：
    ```html
    <ul>
      <li>first</li>
      <li>second</li>
    </ul>

    <ul>
      <li>first</li>
      <li>second</li>
      <li>third</li>
    </ul>
    ```

    React成功匹配两个 `<li>first</li>` ，然后有成功匹配两个 `<li>second</li>` ，下一步发现新树多了 `<li>third</li>` ，于是插入该新的子节点。但是如果我们是这样去插入一个新的节点，性能将会降低：
    ```html
    <ul>
      <li>first</li>
      <li>second</li>
    </ul>

    <ul>
      <li>third</li>
      <li>first</li>
      <li>second</li>
    </ul>
    ```

    React不会按我们的预期那样将 `<li>first</li>` 和 `<li>second</li>` 当作是原有的子节点，它会将三个 `<li>` 都当成新的变化来更新。毕竟当数据量大了之后，这将会是一个很大的性能问题。


那么为了解决这个问题，React提出使用 **`key`** 属性来匹配旧树和新树的子节点。例如：
```html
<ul>
  <li key="1">first</li>
  <li key="2">second</li>
</ul>

<ul>
  <li key="3">third</li>
  <li key="1">first</li>
  <li key="2">second</li>
</ul>
```

这样一来，React在检查到这些子节点时，就知道key为 `"3"` 的节点是新项，而 `"1"` 和 `"2"` 是老朋友了，大家挪个位欢迎新朋友就座就好了。



## Key 如何设置？
- 一般来说，我们会将 `key` 的值设置为数据的「id」。例如 `<li key={todo.id}>{todo.content}</li>` 。
- 如果没有id的话，我们可以为数据模型添加新的id属性，或者对部分内容进行hash处理，以生成每一项的key。
  PS: `key` 仅在它的邻项之间要求唯一，并不具有全局性哦！
- 再不济，我们可以用索引index来作为key值，但是这个方法在一些情况下容易出Bug！例如说在需要重新排序时，由于组件示例的更新和复用是以它们的key为根据的，所以当我们对项目重新排序时， `key` 由于索引的变化也会跟着改变，而诸如非受控组件的状态却不会随之一起改变。官方给出的两个例子就能很好地说明问题：[以index为key](https://reactjs.org/redirect-to-codepen/reconciliation/index-used-as-key) v.s. [以id为key](https://reactjs.org/redirect-to-codepen/reconciliation/no-index-used-as-key) 。
  当然，如果我们将第一个例子中的 `<input/>` 改为受控组件，也可以避免这个问题（送上我改写第一个例子的[传送门](https://codepen.io/caijialinxx/pen/QWWKwVM?editors=0010)）。


---

在长篇大论的铺垫中，终于说完了那么一丁点儿正文。本文就当作是官方文档的一个不对口的翻译吧，如果有理解错误或者不够深入的地方敬请指出～！送上[官方原文](https://reactjs.org/docs/reconciliation.html#keys)。也欢迎大家给我多排除「知识陷阱」～

END.