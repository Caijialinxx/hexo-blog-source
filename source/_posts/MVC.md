---
title: MVC 是什么？
date: 2018-07-08 18:59:58
tags: [Notes]
---

## MVC 是什么

### 我说
> MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。

这是来自[百度百科](https://baike.baidu.com/item/MVC%E6%A1%86%E6%9E%B6/9241230?fr=aladdin&fromid=85990&fromtitle=MVC)的介绍。看上面这段话，我帮你们提取出关键字，再翻译成我们能理解的话，就是：
> MVC 是一种**代码组织形式**，它可以将代码按照 **`Model`** （数据层）、 **`View`** （视图层）和 **`Controller`** （控制层）这三种形式组织代码，每层互相独立、各司其职，却又相互联系、相互依赖。

这样是不是好理解多了，那么这三层的作用如下所述：
- `Model` 负责数据管理，包括数据逻辑、数据请求、数据存储等功能。例如，前端的 Model 主要负责 AJAX 请求或 LocalStorage 的存储
- `View` 负责用户界面的呈现。例如，前端的 View 主要负责 HTML 页面的渲染
- `Controller` 负责处理 View 层的事件，并更新 Model 中的数据；也负责监听 Model 层的数据变化，并及时更新到 View 层。

### 代码说
```js
let model = {
  data: null,
  init() {
    // model 的初始化，一般是要与数据库建立连接，所以包含一些验证信息（id, key）
  },
  fetch() {
    // 获取数据库中的 data(s)
  },
  add() {
    // 向数据库中添加 data(s)
  },
  update() {
    // 更新数据库中的 data(s)
  },
  delete() {
    // 删除数据库中的 data(s)
  }
}

let view = {
  init() {
    // view 获取需要被操作的对象，供 controller 操作
  }
}

let controller = {
  view: null,
  model: null,
  init(view, model) {
    // controller 初始化本地 view 或 model ，并执行相关逻辑操作
    this.view = view
    this.model = model
    this.bindEvents()
  },
  bindEvents() {
    // 处理 view 层的事件，如果数据有变化就更新 model 层
  },
  render() {
    // 监听 model 层的数据变化，更新到 view 层
  }
}

controller.init(view, model)
```

### 图说
由上面的文字和代码，我们可以已经可以知道 `Model` 、 `View` 和 `Controller` 三者的关系了。再用图总结一下，就如下所示：
![图说MVC.png](https://i.loli.net/2018/07/08/5b41e262f3b8d.png)



## MVC 有什么用

WHAT!? 上面的例子还不能让你知道 MVC 的好处吗？

想象一下，你要做了一个网页应用，某一天，某个功能不够好想完善，或者出了 BUG ，假设你把所有 JS 代码全部集中在一个文件里，而且没有整理代码的习惯，是的，你需要在几百行甚至上千行的代码中找到那个功能的代码，.............（不如先给自己准备一瓶眼药水）

好了，不废话了，在你的代码量大的时候，使用 MVC 来组织封装代码，那么在你要找代码的时候，就可以快速定位了。

例如，你想改变某个控件在被点击时的动画效果——好的，这是 `Controller` 层的事，那么直接找到 `Controller` 层中那个控件的 `click` 事件做修改就可以了；

或者你发现同一个文档中有两个拥有相同 `id` 的元素，你已经在 HMTL 中改正，现在只需要去对应的 JS 中改变 `View` 层中使用错误 id 获取的这个元素的正确 id 即可。

...被我绕晕了吗？相信聪明的你，可以看懂，哈哈哈哈哈。


## 总结<small>（其实可以直接看这个）</small>
1. MVC 是一种 
    - 按照 `Model` 、 `View` 和 `Controller` 三层结构
    - 来组织代码
    - 的形式 / 理念 / 典范
2. 它可以
    - 提高代码的可重用性
    - 减少三层结构之间的互相干扰
    - 提高程序的可扩展性和可维护性


---
本文到此结束，谢谢观赏，敬请勘误。
E-mail: caijialinxx@foxmail.com