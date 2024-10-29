---
title: 从 npm 引入 React Hooks 轮子库报错 Minified React error#321 的解决方法
date: 2019-11-20 13:58:24
tags: [Notes, React, webpack]
keywords: hooks, externals, solution, peerDependencies, webpack, npm
---

最近把自己（未完待续）的轮子库 [`cui-demo`](https://github.com/Caijialinxx/ui-demo) 尝试发布到 npm 上，在测试项目中尝试引用时，报了一个无情的错。
![Minified React error #321](https://i.loli.net/2019/11/23/CTO5zkAInMRlt8Q.png)

> 「不看废话版」<br>
> 排除了我的代码问题后，这个报错的原因应该是我的轮子库没有成功获取到测试项目（宿主环境）的依赖 `react` 和 `react-dom` 。解决方法如下：<br>
> 1. 在 webpack 配置中将 `react` 和 `react-dom` 标记为 `externals`（这同时要求 `output.libraryTarget` 为 `umd` ），使轮子库可以在运行时获取到宿主环境的依赖。即
     ```js
     // webpack.config.js
     module.exports = {
       // ...
       externals: {
         react: {
           commonjs: 'react',
           commonjs2: 'react',
           amd: 'react',
           root: 'React',
         },
         'react-dom': {
           commonjs: 'react-dom',
           commonjs2: 'react-dom',
           amd: 'react-dom',
           root: 'ReactDOM',
         },
       }
     };
     ```
> 2. 在 `package.json` 中为 `react` 和 `react-dom` 添加同伴依赖 `peerDependencies` 的映射，这是为了检测宿主环境中这两项依赖的版本如果低于你规定的最低版本，那么在 npm@3 中会给出警告（npm@1 和 npm@2 中会自动安装）。配置如下：
     ```json
     {
       // ...
       "peerDependencies": {
         "react": "^16.8.4",
         "react-dom": "^16.8.4"
       }
     }
     ```

<br>
Debug 开始：

为什么说这个错误提示无情呢？让我们来慢慢走进这个报错世界：
1. 它让我转到 [https://reactjs.org/docs/error-decoder.html/?invariant=321](https://reactjs.org/docs/error-decoder.html/?invariant=321) 去看完整信息。好，我知道这个报错是非法 Hook 调用（Invalid Hook Call）的问题了。

2. 然后我继续看 [https://fb.me/react-invalid-hook-call](https://fb.me/react-invalid-hook-call) 提供的 debug 方法。
    - Breaking the Rules of Hooks
        违反 Hooks 规则的错误，我用我的狗头保证，不是的🐶。
    - Mismatching Versions of React and React DOM
        React DOM 版本与 React 不匹配（低于16.8.0），也排除了。（见下图）
    - Duplicate React
        意外地引入了两个 React ，还是排除了。（见下图）

        ![根据官网进行的debug](https://i.loli.net/2019/11/23/Pjy5US7ap2mvfwu.png)

        而且我的轮子库的 `package.json` 中对 `react` 和 `react-dom` 的版本设置也是符合要求的：
        ```json
        {
          // ...
          "devDependencies": {
            // ...
            "react": "^16.8.4",
            "react-dom": "^16.8.4"
          }
        }
        ```

        在这两个证据之下，我不得不怀疑自己的狗头是不是保不住了... 难不成我真的违反了 Hooks 的规则？？不应该呀... 如果真是这样，那我平时自己写轮子运行的时候就会报错不通过了呀...

3. 带着疑问，我决定去参考 [ant-design](https://github.com/ant-design/ant-design/blob/master/package.json) 和 [element-ui](https://github.com/ElemeFE/element/blob/master/package.json) 的 `package.json` 文件，看看它们是如何配置依赖的。后来发现它们还在 [`peerDependencies`](https://docs.npmjs.com/files/package.json#peerdependencies) 中指定了兼容的版本号。虽说我的轮子库对 `react` / `react-dom` 的依赖宽松到 `>=16.8.4 <17.0.0` ，完全兼容测试项目的 `^16.12.0` ， `peerDependencies` 在这应该不起作用，但为了防止其他用户使用我这个轮子库时宿主环境的依赖版本低于要求，我还是得给 `package.json` 添加这个映射：
    ```diff
    {
      ...
      "devDependencies": {
        ...  
        "react": "^16.8.4",
        "react-dom": "^16.8.4"
      },
    + "peerDependencies": {
    +   "react": "^16.8.4",
    +   "react-dom": "^16.8.4"
    + },
      "dependencies": {}
    }
    ```

    抱着“瞎猫碰上死耗子”的心态，我重新把更改后的轮子库发布到 npm 中。但最后这个更改果然并没有让 `#321` 的报错消失。

    无情展露无余。

<br>

在几次顽强地挣扎和重试后，我真的摸不着头脑了。谷歌了很多篇文章，终于，老天不负有心人，我终于找到了问题所在——原来项目中真的存在着多个 `react` 实例， `cui-demo` 打包时 webpack 把 `react` 一起把它打包进 bundles 里了：

![webpack](https://i.loli.net/2019/11/25/Gl3dDPoxergbiTC.png)

为了解决这个问题，我应该在 webpack 构建时将 `react` 和 `react-dom` 标记为 `externals` （外部扩展），这样它们就不会被打包进 bundles 中，而是在运行时自动从宿主环境中获取这些扩展依赖。配置如下：
```js
// webpack.config.js
module.exports = {
  // ...
  output: {
    path: path.resolve(__dirname, 'dist/lib'),
    library: 'CUI',
    libraryTarget: 'umd'      // 当扩展依赖是对象({ root, amd, commonjs, ... })时，libraryTarget必须为'umd'。查看：https://webpack.js.org/configuration/externals/#object
  },
  externals: {
    react: {
      commonjs: 'react',
      commonjs2: 'react',
      amd: 'react',
      root: 'React',
    },
    'react-dom': {
      commonjs: 'react-dom',
      commonjs2: 'react-dom',
      amd: 'react-dom',
      root: 'ReactDOM',
    },
  }
};
```
![webpack 新增 externals 配置](https://i.loli.net/2019/11/23/XwuHk3a86DVcr2p.png)



好的，再重新发布看看。

YES！终于成功了！在 webpack 中排除依赖打包进外部 bundles 中即可解决我测试项目中 `Minified React error #321` 的报错～戳[此](https://codesandbox.io/s/crazy-https-r9scn?fontsize=14&hidenavigation=1&theme=dark)体验一下新鲜出炉的轮子库 [`cui-demo`](https://github.com/Caijialinxx/ui-demo) 吧！



---
参考资料：
- [Invariant Violation: Minified React error #321 - react issues #16029
](https://github.com/facebook/react/issues/16029#issuecomment-518156374)
- [Externals - webpack](https://webpack.js.org/configuration/externals/) / [中文版](https://webpack.docschina.org/configuration/externals/)
- [Authoring Libraries - webpack](https://webpack.js.org/guides/author-libraries/) / [中文版](https://webpack.docschina.org/guides/author-libraries)
- [npm-package.json peerDependencies](https://docs.npmjs.com/files/package.json#peerdependencies)
- [Peer Dependencies](https://blog.domenic.me/peer-dependencies/) / [Ivan Yan的中文翻译](https://github.com/hongfanqie/peer-dependencies)
- [package.json文件 - JavaScript 标准参考教程（alpha）](https://javascript.ruanyifeng.com/nodejs/packagejson.html#toc3)