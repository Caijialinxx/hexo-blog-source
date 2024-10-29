---
title: TypeScript 的使用心得及笔记
date: 2021-10-02 19:55:04
tags: [TypeScript, Notes]
---

入职这家公司后，项目中大多有使用 TypeScript 进行开发，经过一段时间的使用，我打算为它写写博客。



## TypeScript 是什么，与 JavaScript 有什么区别？
TypeScript 是 JavaScript 的超集，它支持所有 JavaScript 的特性，同时为 JavaScript 提供了类型。也就是说，用一个公示表示即为 `TypeScript = JavaScript + Type` 。 JavaScript 可以直接运行在浏览器和 Node.js 中，但 TypeScript 不行，它需要经过编译转换成 JavaScript 才行。



## 为什么需要 TypeScript ，相比 JavaScript 有什么优势？
JavaScript 从不检查变量的类型，这其实是比较危险的一件事，开发人员编写代码时很容易变得没有“规矩”，运行代码时可能会遇到意料之外的表现或者 Bug 。**正确**、**严格**地使用 TypeScript 可以帮助我们在编写代码时就**避免掉类型或条件判断上的低级错误**，并且 IDE 可以给我们**提供类型甚至代码提示**。

为什么我要强调“正确”、“严格”呢？因为在工作中我就遇到少部分同事因为偷懒不使用类型或者随意使用 `any` ，当我需要查看某一个变量、函数入参或函数返回值等的类型时，往往需要去到声明或定义的地方阅读代码才能确保它的类型到底是哪个(些)，这多少会造成一定的不便及心智负担。

而在遇到规范使用 TypeScript 写出来的代码时，我感觉到心情都舒畅不少。



## 一些特殊的类型
### 顶级类型 `any`
`any` 属于顶级类型（top type），滥用它会使 TypeScript 的作用大打折扣，因为这个类型就是不做任何检查，它允许任何类型的值赋值给 `any` ，同时也可以将 `any` 类型的变量任意赋值给其他有严格类型限制的变量。当然，我们可以在 TypeScript 的编译配置中给 `noImplicitAny` 设置为 `true` ，即可在编译阶段检查到若使用了 `any` 就报错，以限制其使用。

### 另一个顶级类型 `unknown`
与 `any` 相同的是， `unknown` 也是顶级类型，允许任何类型的值赋值给 `unknown` 类型的变量。但不同的是， `unknown` 的类型检查比 `any` 严格，当将 `unknown` 类型的变量赋值给其他严格类型的变量时，会报错：
```ts
const value: unknown = "Hello World";
const someString: string = value;
// 报错：Type 'unknown' is not assignable to type 'string'.(2322)
```

### 与前两者完全相反的 `never`
`never` 是底类型（bottom type），表示不应该出现的类型，任何类型的变量都**不能**赋值给 `never` 类型的变量，并且 `never` 类型的变量也不能赋值给其他类型的变量。对于它的使用，尤雨溪给出了一个[示例](https://www.zhihu.com/question/354601204/answer/888551021)，看看这个例子，我们就能知道它的应用场景是什么了。



## `type` V.S. `interface`
在平时的开发中，你们使用 `type` 还是 `interface` 更多呢？我的使用习惯中，通常对对象的类型定义我是时候后者，而对于前者往往是用在需要用到联合类型（Union Type）的时候。它们两者的区别有以下：
- 前者实际上是创建一个*类型别名*，它不创建一个新的类型，而后者会。
- 前者使用 `&` 来实现类型的扩展，而后者使用 `extends` 实现接口继承。
- 前者不可以多次定义，否则会报错，类型于 `const` ，而后者可以重复定义，并且通过重复定义可实现扩展。
    ```ts
    interface ObjectProps {
      a: string
    }
    interface ObjectProps {
      b: number
    }
    const obj: ObjectProps = { a: '123', b: 123 };

    type Y = string;
    type Y = number;
    // Duplicate identifier 'Y'.(2300)
    ```
- 前者可以直接定义为简单类型，而后者不行。



## 常用的工具类型（未完待续...）
详参考[Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)。
- `Partial<Type>` 、 `Required<Type>`
- `Pick<Type, Keys>` 、 `Omit<Type, Keys>`
- `Exclude<UnionType, ExcludedMembers>` 、 `Extract<Type, Union>`

