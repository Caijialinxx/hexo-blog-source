---
title: JavaScript 中的面向对象（继承）
date: 2019-01-04 18:13:49
tags: [JavaScript, Notes, 继承, 原型]
---

面向对象，是作为一个程序员经常能够碰到的一个概念。那么到底什么是面向对象编程呢？

有人可能会回答：
> 把一组数据结构和处理它们的方法组成对象（object），把相同行为的对象归纳为类（class），通过类的封装（encapsulation）隐藏内部细节，通过继承（inheritance）实现类的特化（specialization）／泛化（generalization），通过多态（polymorphism）实现基于对象类型的动态分派（dynamic dispatch）。
> 如何用一句话说明什么是面向对象思想？ - Milo Yip的回答 - 知乎
> https://www.zhihu.com/question/19854505/answer/23421930

总结确实是已经很精辟了，但对于初学者而言，可能看完就懵逼了，我只是想搞懂面向对象是什么意思，怎么给了我更多的概念出来😭

这大概就是所谓的「从入门到放弃」吧... 哈哈哈哈哈

于是这时有人跑出来回答说：
- 无论是面向对象编程还是函数式编程，这些都只是一种编程思想。就像是不同的宗教，有些不允许养猪吃猪肉，有些不允许吃牛肉，有些要求男性戴帽女性盖头…… 你可以信某个教，也可以不信教或者汲取多个教的精华。也就是说，我们只需要关注面向对象的意义，而不必纠结于一个语言或者程序设计到底是遵循或应该遵循什么编程典范。

<small>这让我想到了咱南北方的饮食差异：北方的粽子是甜的，南方的粽子是咸的；北方人吃完12个饺子后“老板其它菜咋还没上呢”，南方人吃完12个饺子后“老板买单”...😂</small>

那么接下来，我们来详细聊聊 JavaScript 是如何体现面向对象思想的，以及继承该如何实现。



## 封装
我现在想要两百块，那么：
```js
var hundred1 = {
  id: 'AA00000000',
  value: 100,
  unit: '元',
  color: '红',
  form: '纸币',
  issue: '2019-01',
  兑换等值商品: function () {},
  换两张50块: function () {}
}
var hundred2 = {
  id: 'AA00000001',
  value: 100,
  unit: '元',
  color: '红',
  form: '纸币',
  issue: '2019-01',
  兑换等值商品: function () {},
  换两张50块: function () {}
}
```
于是我就可以用这两百块钱去吃椰子鸡了。

### 构造函数 & 原型对象
哎呀，椰子鸡好吃，可是钱也花光了。不行，我得再造钱来用，但我不想让别人看到我是怎么造的！所以我要做一个百元印钞机，把这些过程放百元进印钞机里，我只要输入每张钞票的 ID ，再按印刷键，就可以刷刷刷出百元大钞🤤。同时，这个印钞机要智能一点，不要老是一个个重复得去刻面额、单位、喷颜色等，直接给它引用个百元钞模版，这样能省点墨水😎。

```js
// 百元印钞机——构造函数 Hundred()
function HundredYuan (id) {
  var money = {}
  money.__proto__ = HundredYuan.prototype   // 实际开发中不能这么写，会严重影响性能，详见 MDN 解释：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/proto
  money.id = id
  return money
}
// 百元钞模版——原型对象 HundredYuan.prototype
HundredYuan.prototype = {
  value: 100,
  unit: '元',
  color: '红',
  form: '纸币',
  issue: '2019-01',
  兑换等值商品: function () {},
  换两张50块: function () {}
}

// 给老子来五百，装进钱包里
var wallet = []
for (var i = 2; i < 7; i++) {
  var id = 'AA0000000' + i
  wallet.push(HundredYuan(id))
}

wallet            // 5张100编号分别是 'AA00000002', 'AA00000003', 'AA00000004', 'AA00000005', 'AA00000006'
wallet[0].value   // 100
```

> 这个百元印钞机，就是我们所说的「构造函数」，它能生成一个实例对象（也就是造出一张张百元钞），同时它还用 `prototype` 属性来引用一个存放共同属性的「原型对象」（百元钞模版）。把造钱的过程放进这个百元印钞机，就是「封装」。这样我们就能很好地理解了封装的意义了——隐藏某一方法的具体运行步骤。

### new & this
从上面的例子我们可以知道，构造函数的套路就是：
1. 创建一个空的新对象
2. 为这个新对象添加公有属性，即 `新对象.__proto__` 属性指向构造函数的原型（ `构造函数.prototype` 引用的对象）
3. 为这个新对象添加自有的属性
4. 返回这个新对象。

既然套路都一样，那为什么不把这个套路变得贴心一点呢？于是有了 `new` 操作符，它可以帮我们省去 1、2、4 这三个动作，我们只需专注于添加对象的自有属性即可。同时， JS 之父还帮我们规定好，创建的空对象统一命名为 `this` 。于是，上面的函数我们可以写成这样：
```js
function HundredYuan (id) {
  // var this = {}
  // this.__proto__ = HundredYuan.prototype
  this.id = id
  // return this
}
HundredYuan.prototype = {
  value: 100,
  unit: '元',
  color: '红',
  form: '纸币',
  issue: '2019-01',
  兑换等值商品: function () {},
  换两张50块: function () {}
}

new HundredYuan('AA00000007')
```



## 继承
嘿嘿，有了百元印钞机，富得流油不再是梦🤤。

但是新的问题来了，整天带着一大沓百元大钞出门，要么可能会被乞丐围着，要么可能会有生命危险！不行不行，还是低调点，我需要点小钞。例如五角硬币机：
```js
function FiveJiao () {}
FiveJiao.prototype = {
  value: 5,
  unit: '角',
  color: '金',
  form: '硬币',
  issue: '2019-01',
  兑换等值商品: function () {}
}
```

哇，跟百元印钞机好像哦。说不定把它们整一下，就可以弄出一元印钞机、十元印钞机…… 那么我就弄个造钱机，它造的钱都有 `兑换等值商品` 的功能，我要让 N 元印钞机和 N 角硬币机在不需要自己定义的情况下就能直接用到它爸爸造钱机的 `兑换等值商品` 功能以及一些属性。这就是我们所谓的「继承」——让子类实例能够拥有父类实例的所有方法。
```js
// 父类 Money
function Money(options) {
  this.form = options.form
  this.issue = options.issue
}
Money.prototype.兑换等值商品 = function () { }

// 子类 HundredYuan
function HundredYuan(id) {
  Money.call(this, { form: '纸币', issue: '2019-01' })          // 调用父类 Money 使得子类 HundredYuan 的实例具有父类实例的属性
  this.id = id
}
function tempMoney () { }                     // 1. 为了使 HundredYuan 的原型能够继承父类 Money 的原型，我们造一个空构造函数
tempMoney.prototype = Money.prototype         // 2. 父类 Money 的原型赋给空构造函数的原型
HundredYuan.prototype = new tempMoney()       // 3. 于是 HundredYuan 的原型间接继承父类 Money 的原型
HundredYuan.prototype.value = 100
HundredYuan.prototype.unit = '元'
HundredYuan.prototype.color = '红'
HundredYuan.prototype.换两张50元 = function () { }

// ……以此类推其他印钞机

new HundredYuan('AA00000001')
```

### `Object.create()`
`Object.create()` 方法创建一个新对象，使用现有的对象来提供新创建的对象的 `__proto__` 。 
让子类原型继承父类原型的操作看起来太麻烦了，更人性化及正确的写法如下：
```js
// ...
function HundredYuan(id) {
  Money.call(this, { form: '纸币', issue: '2019-01' })
  this.id = id
}
HundredYuan.prototype = Object.create(Money.prototype)  // 相当于 HundredYuan.prototype.__proto__ = Money.prototype
HundredYuan.prototype.constructor = HundredYuan         // 重新指定原型的 constructor
HundredYuan.prototype.value = 100
// ...
```

### 类
ES6 引入的「类」是一个特殊的函数，它可以帮助我们进一步简化继承的操作，但它依旧是「基于原型」的而不是引入新的面向对象继承模型，类语法只不过是一种语法糖。

上面的例子用类改造如下：
```js
// 父类 Money
class Money {
  constructor (options) {
    this.form = options.form
    this.issue = options.issue
  }
  兑换等值商品 () { }
}

// 子类 HundredYuan
class HundredYuan extends Money {
  constructor (id) {
    super({ form: '纸币', issue: '2019-01' })       // 调用父类 Money 使得子类 HundredYuan 的实例具有父类实例的属性
    this.id = id
    // 在构造函数中的公有属性可以是简单类型，但在类中不可以，所以需要把公有属性放在 constructor 中，作为子类的自有属性。
    this.value = 100
    this.unit = '元'
    this.color = '红'
  }
  换两张50元 () { }
}

new HundredYuan('AA00000001')
```

使用类方法确实简洁明了了许多，但是有个问题就是，它明明是个函数，但却不能被调用：
```js
typeof Money        // "function"
Money()             // Uncaught TypeError: Class constructor Money cannot be invoked without 'new'
```

要用类还是用构造函数，你来决定，这里只是提出这一个现象。

希望看完本文的你，能理解到 JavaScript 中的面向对象思想。


------------
本文完。

若文中有错误还请指正与包涵！

原文链接：https://caijialinxx.github.io/2019/01/04/oop-in-javascript/

转载请注明出处。

参考资料：
- [面向对象编程 - JavaScript 教程 - 阮一峰](https://wangdoc.com/javascript/oop/index.html)


拓展阅读：
- [JS 的 new 到底是干什么的？ - 方应杭的文章 - 知乎](https://zhuanlan.zhihu.com/p/23987456)
- [誓死讲清 prototype 、 __proto__ 、 原型链与继承 - Caijialinxx](https://caijialinxx.github.io/2018/10/22/prototype-proto-and-inheritance/)