---
title: 誓死讲清 prototype 、 __proto__ 、 原型链与继承
date: 2018-10-22 15:24:21
tags: [JavaScript, Notes, 面试, 原型, 继承]
---

尽管看过很多解释这些概念的文章，每次看完都觉得自己理解了。但一遇到这类问题的时候，我还是不免会答错。这次博客的诞生原因，就是今天又做错了题😭，我这破记性，恐怕得写一篇自己的博客梳理一下，才能真正记住并理解。

先给你们看看这篇博客的“罪魁祸首”：
> 现有如下代码：
  ```js
  var F = function() {}
  Object.prototype.a = function() {}
  Function.prototype.b = function() {}
  var f = new F()
  ```
> 问：f.a 和 f.b 都能取到吗？

好的，你们可以先做做这个题。答案我一会儿告诉你们。现在，我要先来讲讲这题涉及到的知识点（其实叫概念更合适？）：
- 构造函数、实例对象与继承
- **原型对象**（如 `Object.prototype`）
- （不知道中文名怎么叫或者根本没有名字的） **`__proto__`**
- 原型链（prototype chain）


## 构造函数、实例对象与继承
构造函数就是用来构造（创建）一个实例对象的函数。举个例子就是：
```js
var 你妈妈 = function() {
  this.带不带把 = '不带把'
  this.笑 = function() { return '咯咯咯' }
  this.撒娇 = function() { return '喵喵喵' }
}
你妈妈.prototype = {
  长相: '美丽动人',
  说话: function() { return '叽里呱啦' },
  笑: function() { return '嘻嘻嘻' }
}
var 你 = new 你妈妈()
```
你就是一个实例对象，你妈妈就是一个构造函数，她把你造出来就是 **new** 了你（不要问我你爸是什么）。你遗传（**继承**）了她的特征（**`你妈妈.prototype`**），例如你妈美丽动人的长相、说话的方法、笑的方法。此外，她期望你是个女孩，笑得可爱，还会撒娇。
于是，你就成了下面这个样子，你需要通过 **`__proto__`** 这根生命线连接你妈妈的原型特征：
<img src="https://i.loli.net/2018/10/22/5bcdaa870a3f6.png" alt="如你妈愿的你" title="如你妈愿的你" />

然而，老天嫉妒你妈脱俗的美貌，决定报复在你的身上，让你变成一个又丑又讨人厌的小孩：
```js
你.长相 = '奇丑无比'
你.嚎啕大哭 = function() { return '哇哇哇' }
```
太可怕了，这真是个悲伤的故事：
<img src="https://i.loli.net/2018/10/22/5bcdaa871af01.png" alt="变了的你" title="变了的你" />

举了个不是非常恰当的例子，戏精该下场了🙂。不过相信聪明的你们应该懂构造函数和实例对象是什么了吧。目前你只需要记住：
> 实例对象被构造（`new`）了之后能够自动引用（`__proto__`）其构造函数的原型对象（`prototype`），同时还可以设置自己的属性或方法，如果与原型对象有同名的属性或方法，则会覆盖原型对象上的属性或方法，优先读取自身的属性或方法。

好了，看完这节，忘记上面只为帮助理解的例子吧...



## 原型对象
原型对象，顾名思义，就是一个对象，它包含着所有实例对象需要共享的属性和方法。下面是一些原型对象包含的属性和方法的举例：
<img src="https://i.loli.net/2018/10/22/5bcdaa86e65d4.png" alt="Object.prototype" title="Object.prototype" />
<img src="https://i.loli.net/2018/10/22/5bcdaa86ecc71.png" alt="Function.prototype" title="Function.prototype" />
<img src="https://i.loli.net/2018/10/22/5bcdaa871e5bc.png" alt="Array.prototype" title="Array.prototype" />

这些原型对象都有一个 `constructor` 属性，它们都指向了自己的构造函数，即
```js
Function.prototype.constructor === Function         // true
Array.prototype.constructor === Array               // true
Object.prototype.constructor === Object             // true
```
<img src="https://i.loli.net/2018/10/22/5bcdaa8728cd3.png" alt="原型对象的constructor指向各自的构造函数" title="原型对象的constructor指向各自的构造函数" />

有没有被绕晕😄，与第一节结合起来看，构造函数里的 `prototype` 是它的原型对象，而这个原型对象里有一个属性 `constructor` 指向构造函数...(??? hahaha~)

如果你还不理解原型对象是什么，那么请先记住：
> 原型对象是一个包含着所有实例对象**需要共享的属性和方法的对象**。



## \_\_proto\_\_
我们在上一节可以看到，这三个原型对象包含的属性和方法不尽相同。例如 `Function.prototype` 有 `call()` 方法，而 `Array.prototype` 和 `Object.prototype` 没有。但 `Function.prototype` 和 `Array.prototype` 都有 `__proto__` 属性，且它们的值指向相同的 `Object.prototype` 。
<img src="https://i.loli.net/2018/10/22/5bcdaa8703644.png" alt="Function.prototype和Array.prototype的__proto__属性指向Object.prototype" title="Function.prototype和Array.prototype的共同属性__proto__属性指向Object.prototype" />

在第一节中，`你._proto__ === 你妈妈.prototype` ，被创建的实例对象的 `__proto__` 指向构造函数的原型对象中。整成一个公式就是：
> **`被构造的对象.__proto__ === 构造函数.prototype`**

那么也就是说 `Function.prototype` 和 `Array.prototype` 都是被 `Object` 构造函数构造出来的。你现在可能已经晕了，没办法理解它们是如何被构造出来的，那么接下来，我又要举造人的例子了（放心，这次保证高级一点），希望能帮助你理解：
```js
// 充当 Object
var 好神仙 = function(性别) {
  this.性别 = 性别
}
好神仙.prototype.constructor = 好神仙
好神仙.prototype.品质 = '善良'
好神仙.prototype.修仙 = function() {}
/* 假装这是控制台：
 * 好神仙.prototype
 * {
 *   + 修仙: ƒ ()
 *     品质: "善良"
 *   + constructor: ƒ (性别)
 *     ...
 * }
 */

// 充当 Function/Array
var 葫芦籽 = function(排行, 颜色, 技能, 弱点) {
  this.排行 = 排行
  this.颜色 = 颜色
  this.技能 = 技能
  this.弱点 = 弱点
}
葫芦籽.prototype = new 好神仙('男')                // 即 Function.prototype = new Object(...)
葫芦籽.prototype.constructor = 葫芦籽
葫芦籽.prototype.叫爷爷 = function() { return '爷爷' }
/* 假装这是控制台：
 * 葫芦籽.prototype
 * {
 *   + constructor: ƒ (排行, 颜色, 技能, 弱点)
 *   + 叫爷爷: ƒ ()
 *     性别: "男"
 *   - __proto__:                                 <-- 指向 好神仙.prototype，相当于 Function.prototype.__proto__ 指向 Object.prototype
 *     + 修仙: ƒ ()
 *       品质: "善良"
 *     + constructor: ƒ (性别)
 *       ...
 * }
 */

葫芦籽.prototype.__proto__ === 好神仙.prototype       // true
```
这里例子中，即使葫芦籽自己有个原型对象 `葫芦籽.prototype` ，这个原型对象也可以通过 `__proto__` 属性指向上一层原型对象 `好神仙.prototype` 。同理 `Function.prototype` 和 `Array.prototype` 。

上面的例子只是为了解释【Function.prototype和Array.prototype的__proto__属性指向Object.prototype】这个是如何实现的，名词多了看起来确实复杂了，好吧看来第一节的例子又要拿出来用了... 我尝试把 `__proto__` 删掉你就能知道它的作用了。
```js
你.说话()                 // "叽里呱啦"
你.__proto__ = null
你.说话()                 // Uncaught TypeError: 你.说话 is not a function
```
看！删了之后你就说不了话了！因为你自身没有说话这个技能，只好靠妈妈助攻，结果你跟她断绝母女关系了！你妈妈伤心欲绝，挥一挥衣袖，带走所有云彩。所以别随便作死😀


说了那么多，整理一下这一节的内容：
> 1. `__proto__` 是实例对象的一个属性，这个属性使得实例对象可以使用其共用的属性和方法。
> 2. 每当一个实例被构造，其 `__proto__` 属性就会自动指向构造它的函数的原型对象。记住这个公式： **`被构造的对象.__proto__ === 构造函数.prototype`** 
> 3. `__proto__` 会一层层向上指，直到遇到 `null` 。

## 原型链
相信看到这里，你应该知道原型链是个啥子东西了。但操心的我还是会举个栗子的（毕竟我的葫芦娃还没放出来呢😀）
```js
var 大娃 = new 葫芦籽(1, '红', {
  力壮术: function() {},
  巨大化: function() {}
}, ['鲁莽', '不懂随机应变'])

大娃.品质                       // "善良"
```
为什么大娃自身明明没有“品质”这个属性，还是可以读到有值呢？

- 因为有原型链呀~！

大娃自身没有，它就是去 `大娃.__proto__` 里找，发现也没有，就继续走到 `大娃.__proto__.__proto__` 里找，大娃长舒一口气：“好家伙总算找到了，原来我的品质是善良呀✌”。所以 `大娃.品质` 实际上是 `大娃.__proto__.__proto__.品质` 。那么大娃走过来的这一路，就是我们所说的“原型链”。
<img src="https://i.loli.net/2018/10/23/5bcef2196c722.png" alt="大娃的原型链" title="大娃的原型链" />


好吧，又得拿出第一节的例子... 我们尝试取你的长相是什么值，结果返回是“奇丑无比”，为什么不是“美丽动人”呢！！？（自己心里没点 B tree 吗）。请看下图，不记得老天报复把你变丑了吗？你在自身属性中找到了你的长相，你看到是“奇丑无比”，天空突然电闪雷鸣、雷雨交加，你哭倒在厕所，无力去走原型链找你的最初的样子，于是一蹶不振接受了这个事实（苍天绕过谁）。
<img src="https://i.loli.net/2018/10/22/5bcdaa871af01.png" alt="变了的你" title="变了的你" />

心情是坏的，但总结还是要做的：
> 当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。  —— [继承与原型链 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)



## 大总结
实例对象被构造了之后，能够自动引用其构造函数的原型对象，即 `实例对象.__proto__ === 构造函数.prototype` 。当试图访问这个实例的一个属性时，它不仅仅会在自身上搜寻，还会沿着它的原型链向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。




好了，戏总有演完的时候。小可爱们是不是都要忘了开头的问题了。
> 现有如下代码：
  ```js
  var F = function() {}
  Object.prototype.a = function() {}
  Function.prototype.b = function() {}
  var f = new F()
  ```
> 问：f.a 和 f.b 都能取到吗？

答案是：不说。自己画原型链分析。不然我这篇博客你就白看了！就是这么善变这么傲娇~

<div style="color: transparent"><p>
f.__proto__ === F.prototype
=> f.__proto__.__proto__ === F.prototype.__proto__ === Object.prototype
=> f.__proto__.__proto__.a
</p><p>
f.constructor === f.__proto__.constructor === F
=> f.constructor.__proto__ === F.__proto__ === Function.prototype
=> f.constructor.__proto__.b
</p></div>


---
本文完。

若文中有错误还请指正与包涵！

原文链接：https://caijialinxx.github.io/2018/10/22/prototype-proto-and-inheritance/

转载请注明出处。

今天的参考资料我想这么写：
- 如果你想了解一下 prototype 等概念的历史，可以看阮一峰老师写的 [Javascript继承机制的设计思想](http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html) ，这篇博客有点年头了，顿时觉得自己落后好多好多好多...
- 如果是初学者的话，建议看 [JS 中 __proto__ 和 prototype 存在的意义是什么？ - 方应杭的回答 - 知乎](https://www.zhihu.com/question/56770432/answer/315342130) ，方老师循序渐进的解答 + 图文并茂的展示，或许能让你理解得更深一些，另外里面也有很多相关文章链接可以点击阅读。
- [对原型、原型链、 Function、Object 的理解](https://zhuanlan.zhihu.com/p/22473059) 这篇文章总结了大量面试常考的关于 `prototype` 与 `__proto__` 的关系的公式，可能可以帮助你轻松记忆。