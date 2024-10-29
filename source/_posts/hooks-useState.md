---
title: useState 的原理及模拟实现 —— React Hooks 系列（一）
date: 2019-12-23 20:49:18
tags: [React, Notes]
keywords: useState, hooks, react
---

> 2023-3-11更新：
> 今天回顾之前写的这篇博客，发现在模拟 useState 实现中，有个很明显的 Bug ，并且有些代码链接也失效了，故更新一版，更新部分会用引用说明，详见下文。


## 基本用法
```jsx
import React, { useState } from "react";
import ReactDOM from "react-dom";

function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

## 原理
按照 React `16.8.0` 版本之前的机制，我们知道如果某个组件是函数组件，则这个 function 就相当于 Class 组件中的 `render()` ，不能拥有自己的状态（故又称其为无状态组件，stateless components），所以数据（输入）必须是来自父组件的 `props` 。而在 `>=16.8.0` 中，函数组件支持通过使用 [Hooks](https://reactjs.org/docs/hooks-intro.html) 来为其引入 state 的能力，例如上面所展示的例子：这个 `App` 组件提供了一个按钮，每次点击这个都会执行 `setCount` 使得 `count` 增加 1 ，并更新在视图上。

为了更直观地看到使用了 Hooks 的函数组件和 Class 组件的区别，我写了一个[~~对比示例~~](https://codesandbox.io/embed/pedantic-grothendieck-bmqgz?fontsize=14&hidenavigation=1&theme=dark)。

> 2023-3-11更新：
> 上方的代码链接失效了，我直接将代码贴在下方：
> 
>     import React, { Component, useState } from "react";
>     import ReactDOM from "react-dom";
>     
>     const logNewValueAfterOneSecond = (c, value) => console.log(`1 second later, ${c}’s count:`, value);
>     
>     const FunctionComponent = () => {
>       const [count, setCount] = useState(0);
>       console.log("A’s count:", count);
>       return (
>         <p>
>           <span>A: {count}</span>
>           <button
>             onClick={() => {
>             setCount(count + 1);
>             setTimeout(() => logNewValueAfterOneSecond("A", count), 1000);
>             }}
>           >
>             A+1
>           </button>
>         </p>
>        );
>      };
>     
>     class ClassComponent extends Component {
>       constructor() {
>         uper();
>         this.state = { count: 0 };
>       }
>       render() {
>         console.log("B’s count:", this.state.count);
>         return (
>           <p>
>             <span>B: {this.state.count}</span>
>             <button
>               onClick={() => {
>                 this.setState({ count: this.state.count + 1 });
>                 setTimeout(
>                   () => logNewValueAfterOneSecond("B", this.state.count),
>                   1000
>                 );
>               }}
>             >
>               B+1
>             </button>
>           </p>
>         );
>       }
>     }
>     
>     function App() {
>       return (
>         <div>
>           <FunctionComponent />
>           <ClassComponent />
>         </div>
>       );
>     }
>     
>     const rootElement = document.getElementById("root");
>     ReactDOM.render(<App />, rootElement);
> 
> 新的代码链接：[Function Component vs Class Component](https://codesandbox.io/s/function-component-vs-class-component-zfybbl)

`A` 是使用了 Hooks 的函数组件， `B` 是我们熟悉的 Class 组件。当我们分别点击两个按钮更新数据时，我们已知 `B` 组件不会重新调用而是在 `setState` 通知变化之后重新渲染（render），而 `A` 组件是如何处理更新的呢？让我们来分析一下这个过程：
- A. 点击按钮「A+1」
    1. 按钮「A+1」的 click 事件触发，执行异步的 `setCount(count + 1)`
    2. 开启定时器，将在一秒之后执行回调函数 `() => logNewValueAfterOneSecond('A', count)`
    3. 按钮「A+1」的 click 事件执行到结尾，异步 `setCount` 起效，输出“*A's count: 1*”， `<span>0</span>` 更新为 `<span>1</span>` （只更新 `span` 的文本）
    4. 一秒到了，定时器的回调函数执行，输出“*1 second later, A's count: 0*”
        
- B. 点击按钮「B+1」
    1. 按钮「B+1」的 click 事件触发，执行异步的 `this.setState(state => ({ count: state.count + 1 }))`
    2. 开启定时器，将在一秒之后执行回调函数 `() => logNewValueAfterOneSecond('B', this.state.count)`
    3. 按钮「B+1」的 click 事件执行到结尾，异步 `setState` 起效，输出“*B's count: 1*”且 `<span>0</span>` 更新为 `<span>1</span>` （只更新 `span` 的文本）
    4. 一秒到了，定时器的回调函数执行，输出“*1 second later, B's count: 1*”

![Hooks+函数组件 V.S. Class组件](https://i.loli.net/2019/12/23/pSEFYXhRcQmNtur.gif)

通过上面的步骤分析，我们可以看到，有区别的就是步骤 **1**、**4** 。先说说 B ——在 B.1 中 `setState` 通知变化后， React 不会立即更新组件，而是会延迟调用它，到了步骤 3 时 React 对 state 的变更才真正生效（类似于 `Object.assign(previousState, {count: state.count + 1})`）， `this.state.count` 的值更新为 1 ，且由于 `shouldComponentUpdate()` 默认返回 `true` ，所以成功触发新的一次渲染，于是组件 `B` 的 DOM 更新（不清楚组件更新过程的同学可以戳[这里](https://reactjs.org/docs/react-component.html)补功课，重点放在[生命周期](https://zh-hans.reactjs.org/docs/react-component.html#the-component-lifecycle)和[setState](https://zh-hans.reactjs.org/docs/react-component.html#setstate)两个模块）。整个过程没有重新生成这个组件实例，它们始终在同一个环境，所以一秒后输出 `B` 的 `count` 是 *1* ，而 A ：
1. 首先， `A` 中的 `count` 和 `setCount` 是由钩子函数 `useState()` 返回的两个值。
2. `count` 的初始值是 0 ，与我们赋予的一样。
3. 上面我说到，函数组件相当于 Class 组件中的 `render()` ，所以我们大胆假设在按钮第一次点击异步执行 `setCount(count + 1)` 之后，重新 render ，即 `A()` 被重新调用，此时 `count` 从 `useState()` 中再次获取到的值更新为 1 ，返回的值通过 diff 最终局部更新。

但是奇怪的是，为什么一秒之后定时器回调函数执行后输出的 `count` 是旧的值呢？请大家再看一遍 3 ，「`A()` 被重新调用」，「`count` 从 `useState()` 中再次获取值」，那么此时的 `count` 还为彼时的 `count` 吗？答案当然是「不」！它们已经是存在于不同环境的两个变量了。
假设 2 中的 `count` 是 count0（值为0），而 3 中的 `count` 是 count1（值为1）。定时器是什么时候设定的？在第一次点击按钮时。这个时候 `logNewValueAfterOneSecond()` 传入的 `count` 实际上是还没有更新的 count0 ，一秒之后，虽然 `count` 已更新为 count1 ，但 `logNewValueAfterOneSecond()` 调用的 `count` 并没有更新，还是之前的 count0 ，所以必然输出 0 。
这下看起来明了了一些吗？接下来我们来尝试模拟实现一下 `useState()` 吧～


## 模拟实现
`React.useState()` 里都做了些什么：
1. 将初始值赋给一个变量我们称之为 `state`

2. 返回这个变量 `state` 以及改变这个 `state` 的回调函数我们称之为 `setState`

3. 当 `setState()` 被调用时， `state` 被其传入的新值重新赋值，并且更新根视图
    ```js
    function useState(initialState) {
      let _state = initialState;
      const setState = (newState) => {
        _state = newState;
        ReactDOM.render(<App />, rootElement);
      };
      return [_state, setState];
    }
    ```

4. 每次更新时，函数组件会被重新调用，也就是说 `useState()` 会被重新调用，为了使 `state` 的新值被记录（而不是一直被重新赋上 `initialState`），需要将其提到外部作用域声明
    ```js
    let _state;
    function useState(initialState) {
      _state = _state === undefined ? initialState : _state;
      const setState = (newState) => {
        _state = newState;
        ReactDOM.render(<App />, rootElement);
      };
      return [_state, setState];
    }
    ```

    好的，现在让我们来调用一下，目前暂时是达到了 `React.useState()` 一样的效果。[Open in CodeSandbox](https://codesandbox.io/s/elegant-dust-5x1oc?fontsize=14&hidenavigation=1&theme=dark)
    ![模拟 React.useState 1.0](https://i.loli.net/2019/12/23/M3hwZkCTgVLD6Gp.gif)

5. 但是，如果添加多个 `useState()` ，就一定会出现 BUG 了。因为当前的 `_state` 只能存放一个单一变量。如果我将 `_state` 改成数组存储呢？让这个数组 `_state` 根据当前操作 `useState()` 的索引向内添加 state
    ```js
    let _state = [], _index = 0;
    function useState(initialState) {
      let curIndex = _index; // 记录当前操作的索引
      _state[curIndex] = _state[curIndex] === undefined ? initialState : _state[curIndex];
      const setState = (newState) => {
        _state[curIndex] = newState;
        ReactDOM.render(<App />, rootElement);
        _index = 0; // 每更新一次都需要将_index归零，才不会不断重复增加_state
      }
      _index += 1; // 下一个操作的索引
      return [_state[curIndex], setState];
    }
    ```

    虽然通过使用数组存储 `_state` 成功模拟了多个 `useState()` 的情况（[Open in CodeSandbox](https://codesandbox.io/s/frosty-flower-9i1y7)），但这要求我们保证 `useState()` 的调用顺序，所以我们不能在循环、条件或嵌套函数中调用 `useState()` ，这在 `React.useState()` 同样要求，官网还给出了专门的[解释](https://reactjs.org/docs/hooks-rules.html#explanation)。

6. 让我们的 `setState()` 也支持[函数式更新](https://reactjs.org/docs/hooks-reference.html#functional-updates)和[惰性初始state](https://reactjs.org/docs/hooks-reference.html#lazy-initial-state)：[Open in CodeSandbox](https://codesandbox.io/s/pedantic-flower-cbyfg)

7. 通过 [`Object.is()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较算法来判断 `_state` 是否需要更新：[Open in CodeSandbox](https://codesandbox.io/s/bold-fire-jv9y7)

### 2023-3-11更新
> 细心的同学可能在第4步是就已经发现问题了——判断 `_state` 是初始赋值还是处于更新阶段时，上文使用的条件是 `_state === undefined ? initialState : _state[curIndex]` 。那么这样就会导致一个问题——当我想要更新的值就是 `undefined` 时，岂不是会被判定为是“初始赋值”并被更新值为传入的 `initialState` 而非当前想要的结果。
> 那么如何把这个条件判断写得更合理呢？结合第5步，我们修改代码如下：
> 
>     // 使用hasOwnProperty去判断当前索引是否被创建，如果有这个索引，说明已经对这个state赋初始值了，如果没有，则说明是新创建的，需要使用initialState赋初始值。
>     _state[curIndex] = !_state.hasOwnProperty(curIndex) ? initialState : _state[curIndex];
> 
> 最终版代码：[useState final version - Open in CodeSandbox](https://codesandbox.io/s/usestate-final-version-0icns7)

至此，我的模拟实现已结束。而实际上， React 并不是真的是这样实现的。上面提到的 `_state` 其实对应 React 的 `memoizedState` ，而 `_index` 实际上是利用了链表。有兴趣的同学可以自己去读[源码](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberHooks.js)或者参阅这篇[博客](https://juejin.im/post/5bdfc1c4e51d4539f4178e1f)。


## 总结
```js
const [state, setState] = React.useState(initialState);
```
- React 16.8.0 正式增加了 Hooks ，它为函数组件引入了 state 的能力，换句话说就是使函数组件拥有了 Class 组件的功能。
- `React.useState()` 返回的第二个参数 `setState` 用于更新 `state` ，并且会触发新的渲染。同时，在后续新的渲染中 `React.useState()` 返回的第一个 `state` 值始终是最新的。
- 为了保证 `memoizedState` 的顺序与 `React.useState()` 正确对应，我们需要保证 Hooks 在**最顶层调用**，也就是**不能**在循环、条件或嵌套函数中调用。
- `React.useState()` 通过 [`Object.is()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 来判断 `memoizedState` 是否需要更新。


---

参考资料：
- [Hooks API Reference - useState](https://reactjs.org/docs/hooks-reference.html#usestate)
- [Rules of Hooks](https://reactjs.org/docs/hooks-rules.html)
- [Introducing Hooks](https://reactjs.org/docs/hooks-intro.html)
- [阅读源码后，来讲讲React Hooks是怎么实现的 - Jokcy](https://juejin.im/post/5bdfc1c4e51d4539f4178e1f)

以上 React 的参考资料的中文版直接在前面加“zh-hans.”即可，即 https://zh-hans.reactjs.org/... ，不过似乎要翻墙才可以😂