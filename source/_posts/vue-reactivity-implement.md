---
title: Vue 双向绑定的实现原理
date: 2019-02-17 14:10:34
tags: [Vue, JavaScript, 面试]
---

我们都知道， Vue 最大的特点之一就是响应式的双向绑定。那么它的实现原理是怎么样的呢？无论是深入学习 Vue 框架也好，还是作为一个面试常考题也好，这都是前端必须了解的一个问题。那么我们今天就来探索一下它的实现方法。


## 模拟实现
Vue 在它的官网中就已经有对它的这个「双向绑定」特性进行说明。详情请戳[传送门](https://cn.vuejs.org/v2/guide/reactivity.html)。我们可以看到， Vue 会遍历实例的 `data` 对象的所有属性，并使用 `Object.defineProperty` 把这些属性全部转为 `getter` / `setter`。根据 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 的介绍，我尝试着使用这个方法来做一个简单的模拟实现：
```js
var $person1 = {},   // 模拟 vm.$data
  person1 = { name: 'caaa' }  // 模拟我们在 Vue 中的 data 选项

Object.defineProperty($person1, 'name', {
  get() { return person1.name },
  set(val) {
    person1.name = val
    // 执行某些操作实现 DOM 局部更新
    console.log('发生了更新')
  }
})

$person1.name             // 'caaa'
$person1.age              // undefined

/* 改变 $person1.name */
$person1.name = 'jack'    // '发生了更新'
$person1.name             // 'jack'
person1.name              // 'jack'
```


## 局限
通过上面的例子，我们知道了 Vue 是如何实现响应式原理的，但是「受现代 JavaScript 的限制，Vue 不能检测到对象属性的添加或删除。」例如说，我们对上面的 `$person1` 添加一个属性：
```js
/* 为 $person1 添加 age 属性 */
$person1.age = 18
$person1.age              // 18
person1.age               // undefined
```

我们可以看到，即使已经改变了 `$person1.age` ， `person1.age` 也依旧没有变化。这是因为 `$person1.age` 是一个非响应的属性，它并没有 `setter` 来对它进行追踪。也就是说 `$person1.age = 18` 其实等同于：
```js
Object.defineProperty($person1, 'age', {
  value: 18,
  enumerable: true,
  writable: true
})
```

那么要如何在 Vue 中为已创建的实例动态添加新的根级响应式属性呢？ Vue 提供的一个方法是 `Vue.set(object, key, value)` 。这里不详细举例说明，有兴趣的可以自己去尝试一下。

我想展开讲的是， Vue 3.0 中对于响应式数据的更新。

⬇️

⬇️

⬇️

## Proxy
尤雨溪老师在 VueConf TO 大会上发表了「Vue 3.0 Updates」的演讲，提到了 3.0 版本将使用原生 [`Proxy`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 来取代 `Object.defineProperty` ，这使得 Vue 可以跳出无法添加或删除响应式属性的局限。那么废话少说，我们立马来用用这聪明的 `Proxy` ：
```js
var person2 = { name: 'caaa' }

var $person2 = new Proxy(person2, {
  get(target, key) {
    return target[key]
  },
  set(target, key, newVal) {
    console.log(`更新了${key}`)
    target[key] = newVal
  }
})

$person2.name           // 'caaa'
$person2.age            // undefined

/* 改变 $person2.name */
$person2.name = 'jack'  // '更新了name'
$person2.name           // 'jack'
person2.name            // 'jack'

/* 为 $person2 添加 age 属性 */
$person2.age = 18       // '更新了age'
$person2.age            // 18
person2.age             // 18

/* 为 $person2 删除 age 属性 */
delete $person2.age     // true
person2.age             // undefined
```

看， `person2.age` 随着 `$person2.age` 的改变而作出相同的改变了！这说明 `$person2` 直接新增的属性也可以是响应式的了～

有点期待 Vue 3.0 的问世了呢🤤 当然 3.0 还有很多其他更新呢，给你[传送门](https://docs.google.com/presentation/d/1yhPGyhQrJcpJI2ZFvBme3pGKaGNiLi709c37svivv0o/edit#slide=id.p)，带你去看「Vue 3.0 Updates」。



本文的演示请点击[传送门](https://jsbin.com/jalunicoya/edit?js,console,output)查看 AND 捣鼓。

---
本文完。

若文中有错误还请指正与包涵！

原文链接：https://caijialinxx.github.io/2019/02/17/vue-reactivity-implement/

转载请注明出处。

参考资料：
- [深入响应式原理](https://cn.vuejs.org/v2/guide/reactivity.html)
- [Object.defineProperty | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
- [Proxy | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)