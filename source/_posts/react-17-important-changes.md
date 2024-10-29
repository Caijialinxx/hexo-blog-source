---
title: React 17 的重要更新
date: 2022-05-02 20:07:44
tags: [React, 面试]
---

React 17 版本没有产生新的特性，是一个为了后续版本升级更方便、安全的垫脚石。其中比较重要升级点如下：
- [事件委托](https://legacy.reactjs.org/blog/2020/08/10/react-v17-rc.html#changes-to-event-delegation)。从挂在到 `document` 上，变更为挂在到 **rootNode** （React应用挂载的根节点）上。
    ![React 17 的事件委托](https://legacy.reactjs.org/static/bb4b10114882a50090b8ff61b3c4d0fd/31868/react_17_delegation.png)
    这使得当存在多个不同版本的 React 树嵌套，或者嵌入到其他程序构建的应用程序中时，不会破坏 `e.stopPropagation()` 。
    另外，React 17 的事件传递更接近原生 DOM 。具体表现为在 React 16 及更早版本中，即使你在 React 事件中调用了 `e.stopPropagation()` ， `document` 监听器器仍然会接收到，这是因为原生事件已经处于 `document` 层级了。在 React 17 中传递将会按要求停止，不会再触发 `document` 对应的事件。
- [新的 JSX 转换](https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)。与 Babel 合作，提供一种全新版本的 JSX 转换方式——无需额外导入 React 。
    ```jsx
    import React from 'react';

    function App() {
      return <h1>Hello World</h1>;
    }
    ```
    旧的 JSX 转换方式会将它转成下列代码：
    ```js
    import React from 'react';

    function App() {
      return React.createElement('h1', null, 'Hello world');
    }
    ```
    而在 React 17 中，源码可以写成这样：
    ```jsx
    function App() {
      return <h1>Hello World</h1>;
    }
    ```
    新的 JSX 转换方式会将它转成下列代码：
    ```js
    // Inserted by a compiler (don't import it yourself!)
    import {jsx as _jsx} from 'react/jsx-runtime';

    function App() {
      return _jsx('h1', { children: 'Hello world' });
    }
    ```
    在升级到 React 17 过程中，如果想批量移除无用的 React 导入，可以在你的项目中执行 `npx react-codemod update-react-imports` ，即可批量移除：
    ```jsx
    import React from 'react';

    function App() {
      return <h1>Hello World</h1>;
    }
    ```
    将会被替换成
    ```jsx
    function App() {
      return <h1>Hello World</h1>;
    }
    ```
    如果使用了其他如 Hook 的 React 导入：
    ```jsx
    import React from 'react';

    function App() {
      const [text, setText] = React.useState('Hello World');
      return <h1>{text}</h1>;
    }
    ```
    将会被替换成
    ```jsx
    import { useState } from 'react';

    function App() {
      const [text, setText] = React.useState('Hello World');
      return <h1>{text}</h1>;
    }
    ```
- [移除了事件池](https://legacy.reactjs.org/blog/2020/08/10/react-v17-rc.html#no-event-pooling)。
    React 在旧浏览器中为了性能而重用不同事件之间的事件对象，并在它们之间将所有事件字段设置为null。在 React 16 及更早版本，你必须调用 `e.persist()` 才能正确使用事件，否则需要提前读取需要的属性。而现代浏览器中，这个性能优化并没有作用，所以 React 17 已经彻底移除了“事件池”
- [异步执行 `useEffect` 清理函数](https://legacy.reactjs.org/blog/2020/08/10/react-v17-rc.html#effect-cleanup-timing)。
    此前 `useEffect` 在组件卸载时，是同步执行的，类似于 `componentWillUnmount` ，这对于大型应用程序来说并不理想，因为它会减慢大屏过渡的速度。在 React 17 中这个过程将会变成异步执行，即在更新已渲染完成后执行。若想要保持同步执行，那么可以使用 `useLayoutEffect` 来代替。当然，不必担心的是， React 17 会保证在执行任何新的副作用之前执行完所有副作用的清理函数。
    当然这个改动会有潜在的问题，当组件中使用了 ref 来执行一些清理操作，如下代码所示：
    ```js
    useEffect(() => {
      someRef.current.someSetupMethod();
      return () => {
        someRef.current.someCleanupMethod();
      };
    });
    ```
    由于 ref 的值是可变的，所以在异步调用时，这个 ref 的值已经被设置为 `null` 了，也就意味着这个清理函数并无法执行。那么解决方案除了使用上述的 `useLayoutEffect` 同步执行以外，还可以这样：
    ```js
    useEffect(() => {
      const instance = someRef.current;
      instance.someSetupMethod();
      return () => {
        instance.someCleanupMethod();
      };
    });
    ```
    将 ref 的值的引用赋值到一个新的变量，那么在清理函数执行时，可以保证这个引用还存在在内存中，可以调用到里面的方法。
- [`render` 中返回 `undefined` 时一致的报错表现](https://legacy.reactjs.org/blog/2020/08/10/react-v17-rc.html#consistent-errors-for-returning-undefined)。
    此前， React 只在 class 组件或函数组件中做了“返回 `undefined`”的检查，错误遗漏了 `forwardRef` 和 `memo` 组件， React 17 中已经将这个错误修复了。如果有意返回空内容，可以返回 `null` 。
- [更好的组件堆栈跟踪](https://legacy.reactjs.org/blog/2020/08/10/react-v17-rc.html#native-component-stacks)。报错时，不再是简单的 JS 堆栈，而是可以显示出具体的 React 组件堆栈信息，并可以快速定位到。