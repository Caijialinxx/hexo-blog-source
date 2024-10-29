---
title: 一些经典的JS手写函数实现
date: 2021-02-21 17:10:26
tags: [Notes, JavaScript, Promise, 深拷贝, debounce, throttle, AJAX]
---

> 以下手写函数的测试/调试地址：https://jsbin.com/ruyukifofu/4/edit?js,console

## 防抖节流
```js
/*
 * 防抖：在xx时间内连续触发不执行，直至停止操作后的xx时间后才执行一次。
 * 假设一个点代表一个时间间隔，防抖设置等待5个时间间隔，#表示一个时间间隔内触发一次动作，!表示实际触发结果：
 * ##··#·#·······##······
 * ············!········!
 */
const debounce = (fn, delay) => {
  let timer = null;
  return (...args) => {
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(undefined, args);
    }, delay);
  };
};

/*
 * 节流：第一次触发执行后，需等待xx时间后才能再次触发，期间连续触发不执行。
 * 假设一个点代表一个时间间隔，节流设置等待5个时间间隔，#表示一个时间间隔内触发一次动作，!表示实际触发结果：
 * ##··#·#·······##······
 * !·····!·······!·······
 */
const throttle = (fn, delay) => {
  let timer = null;
  return (...args) => {
    if (!timer) {
      fn.apply(undefined, args);
      timer = setTimeout(() => {
        timer = null;
      }, delay);
    }
  };
};

// 测试代码
const fn = debounce((...args) => console.log(args), 2000);
const fn2 = throttle((...args) => console.log(args), 2000);
let intervalCount = 0;
const timerId = setInterval(() => {
  intervalCount += 1;
  console.log(intervalCount);
  fn1("debounce", intervalCount);
  fn2("throttle", intervalCount);
  if (intervalCount >= 10) {
    clearTimeout(timerId);
  }
}, 1000);
```


## Promise
```js
class Promise {
  #PromiseState = "pending";
  #PromiseResult = undefined;
  constructor(fn) {
    const resolve = (response) => {
      this.#PromiseState = "fulfilled";
      this.#PromiseResult = response;
    };
    const reject = (reason) => {
      this.#PromiseState = "rejected";
      this.#PromiseResult = reason;
    };
    fn.call(this, resolve, reject);
  }
  then(onfulfilled, onrejected) {
    return new Promise((resolve, reject) => {
      try {
        let result;
        if (this.#PromiseState === "fulfilled" && typeof onfulfilled === "function") {
          result = onfulfilled(this.#PromiseResult);
        } else if (this.#PromiseState === "rejected" && typeof onfulfilled === "function") {
          result = onrejected(this.#PromiseResult);
        }
        resolve(result);
      } catch (e) {
        reject(e);
      }
    });
  }
  catch(onrejected) {
    return new Promise((resolve, reject) => {
      try {
        resolve(onrejected(this.#PromiseResult));
      } catch (e) {
        reject(e);
      }
    });
  }
}
Promise.resolve = (data) => new Promise((resolve) => resolve(data));
Promise.reject = (data) => new Promise((_, reject) => reject(data));
Promise.all = (promises) => {
  const result = [];
  let fulfilledCount = 0;
  return new Promise((resolve, reject) => {
    promises.forEach((p, i) => {
      if (!(p instanceof Promise)) {
        fulfilledCount += 1;
        result[i] = p;
      } else {
        p.then(
          (data) => {
            fulfilledCount += 1;
            result[i] = data;
            if (fulfilledCount >= promises.length) {
              resolve(result);
            }
          },
          (data) => {
            reject(data);
          }
        );
      }
    });
  });
};

// 测试代码
var p1 = new Promise((resolve, reject) => resolve(1.1));
p1.then((data) => { console.log(2.1, data); return 2.1; }, (data) => { console.log(2.2, data); return 2.2; }) // => 2.1 1.1
  .then((data) => { console.log(3.1, data); return 3.1; }, (data) => { console.log(3.2, data); return 3.2; }); // => 3.1 2.1

var p2 = new Promise((resolve, reject) => reject(1.2));
p2.then((data) => { console.log(2.1, data); return 2.1; }, (data) => { console.log(2.2, data); throw 2.2; }) // => 2.2 1.2
  .then((data) => { console.log(3.1, data); return 3.1; }, (data) => { console.log(3.2, data); return 3.2; }); // => 3.2 2.2

var p3 = new Promise((resolve, reject) => resolve(1.1));
p3.then((data) => { console.log(2.1, data); throw 2.1; }, (data) => { console.log(2.2, data); return 2.2; }) // => 2.1 1.1
  .then((data) => { console.log(3.1, data); return 3.1; }, (data) => { console.log(3.2, data); return 3.2; }); // => 3.2 2.1

var p4 = new Promise((resolve, reject) => resolve(1.1));
p4.then((data) => { console.log(2.1, data); throw 2.1; }) // => 2.1 1.1
  .catch((data) => { console.log("catch", data); throw "catch error"; }); // catch 2.1

Promise.all([1, Promise.resolve(2)]).then((values) => { console.log(values); }); // [1,2]
Promise.all([1, Promise.reject('err')]).catch((err) => { console.log(err); }); // 'err'
```


## 深拷贝
```js
// 1.类型判断   2.递归   3.检查环
const deepClone = (originalData, cache) => {
  if (!cache) {
    cache = new Map();
  }
  if (originalData instanceof Object) {
    if (cache.get(originalData)) {
      // 检查环，如果已经有拷贝过这个对象，则直接返回
      return cache.get(originalData)
    }
    let result
    if (originalData instanceof Function) {
      // 函数
      if (originalData.prototype) {
        // function() {}
        result = function () { originalData.apply(this, arguments) }
      } else {
        // 箭头函数
        result = (...args) => { originalData.apply(undefined, args) }
      }
    } else if (originalData instanceof Array) {
      // 数组
      result = [];
    } else if (originalData instanceof Date) {
      // 日期
      result = new Date(originalData - 0);
    } else if (originalData instanceof RegExp) {
      // 正则
      result = new RegExp(originalData.valueOf());
    } else if (typeof originalData.valueOf() !== 'object') {
      // 构造函数new出来的boolean、number、string
      result = new originalData.constructor(originalData.valueOf())
    } else {
      // 普通对象
      result = {};
    }
    cache.set(originalData, result); // 为当前处理的对象cache当前结果，需要递归执行前，否则递归进入后无法获取到cache
    for (let key in originalData) {
      result[key] = deepClone(originalData[key], cache);
    }
    return result;
  } else if (typeof originalData === 'symbol') {
    return Symbol(originalData.description)
  } else {
    // boolean,string,number,undefined,null,bigint
    return originalData
  }
}

// 测试函数
deepClone(123); // 123
deepClone(Symbol('this is a symbol')); // Symbol(this is a symbol)
deepClone(BigInt(12345678901234567890)); // 12345678901234567168n
const obj = {
  num: NaN,
  txt: 'text',
  bool: new Boolean(1),
  symb: Symbol('symbol'),
  fn: (v1, v2) => { console.log('箭头函数', this, v1, v2); }, // this->window
  obj: {
    fn: function(v1, v2) { console.log('普通函数', this, v1, v2);  }, // this->obj
    arr: [1, null, { a: [3], b: { c: 7 } }, BigInt(13245), undefined, false],
    date: new Date('1996-7-8'),
    num: Number()
  },
  reg: new RegExp(/\s+/g)
}
obj.self = obj
const clonedObj = deepClone(obj);
clonedObj.self === clonedObj; // true
obj.obj.arr[0] = 0;
clonedObj.obj.arr[0]; // 1
```


## 数组去重
```js
// 实现一：
const uniq = (array) => {
  const map = new Map();
  array.forEach((item) => {
    map.set(item, 1);
  });
  return [...map.keys()];
}

// 实现二
const uniq2 = (array) => [...new Set(array)];
```


## AJAX
```js
const xhr = new XMLHttpRequest();
xhr.open("GET", "/archives/");
xhr.onreadystatechange = () => {
  if (xhr.readyState === 4) {
    if (xhr.status >= 200 && xhr.status < 300) {
      console.log(xhr.response);
    }
  }
};
xhr.send();
```


## 发布订阅
```js
const EventHub = {
  eventMap: {}, // 映射
  on: (type, fn) => {
    EventHub.eventMap[type] = EventHub.eventMap[type] || [];
    EventHub.eventMap[type].push(fn);
  },
  emit: (type, ...args) => {
    if (EventHub.eventMap[type]) {
      EventHub.eventMap[type].forEach((fn) => {
        fn.apply(undefined, args);
      });
    }
  },
  off: (type, fn) => {
    const queue = EventHub.eventMap[type];
    if (queue && queue.includes(fn)) {
      const index = queue.indexOf(fn);
      queue.splice(index, 1);
    }
  },
};

// 测试代码
const fn = (...args) => console.log("xxx emit", ...args);
EventHub.on("xxx", fn);
setTimeout(() => {
  EventHub.emit("xxx", "jialin", "cai");
  EventHub.off("xxx", fn);
}, 3000);
setTimeout(() => {
  EventHub.emit("xxx", "caijialinxx");
}, 4000);
```

---

## 事件委托
```js
function delegate(selector, eventType, fn) {
  document.addEventListener(eventType, (e) => {
    let target = e.target;
    while(!target.matches(selector)) {
      target = target.parentElement;
      if(target === document.documentElement) {
        target = null;
        break;
      }
    }
    if(target) {
      fn(target)
    }
  })
}
```


## 可拖拽的元素
凑数用的，手写一个可拖拽的元素。预览链接：https://codepen.io/caijialinxx/pen/JjaMjVm
这个加上了边框计算：https://jsbin.com/bikojavuva/1/edit?css,js,output
```html
<div class="box">
  <div class="dragger"></div>
</div>
```
```css
.box {
  width: 400px;
  height: 400px;
  background-color: pink;
  position: relative;
  box-sizing: border-box;
  margin-top: 100px;
  margin-left: 50px;
}

.dragger {
  border-radius: 50%;
  width: 20px;
  height: 20px;
  border: 1px solid red;
  cursor: pointer;
  position: absolute;
  top: 0px;
  left: 0px;
  box-sizing: border-box;
}
```
```js
const dragger = document.querySelector('.dragger');
const box = document.querySelector('.box');

let draggable = false;
let x, y; // 记录当一次鼠标的坐标
const boxWidth = box.offsetWidth;
const boxHeight = box.offsetHeight;
const offsetX = box.offsetLeft; // dragger可移动范围的横向偏移量
const offsetY = box.offsetTop; // dragger可移动范围的纵向偏移量
const maxLeft = boxWidth - dragger.offsetWidth;
const maxTop = boxHeight - dragger.offsetHeight;

// 帮助函数：获取范围内的值
const getValueInRange = (val, min, max) => {
  if(val < min) return min;
  if(val > max) return max;
  return val
}

// 鼠标点击拖动目标时设置draggable为true并记录当前鼠标坐标。
dragger.onmousedown = (e) => {
  draggable = true;
  x = e.clientX;
  y = e.clientY;
}

// 在document上监听鼠标移动事件
document.onmousemove = (e) => {
  if(!draggable) return; // 当draggable为false时不执行拖动逻辑
  const deltaX = e.clientX - x; // 计算横向偏移量
  const deltaY = e.clientY - y; // 计算纵向偏移量
  let left = parseFloat(dragger.style.left || 0) + deltaX; // 计算当前dragger的left值
  let top = parseFloat(dragger.style.top || 0) + deltaY; // 计算当前dragger的top值
  
  // 设置dragger的left/top在规定范围内
  dragger.style.left = getValueInRange(left, 0, maxLeft) + 'px';
  dragger.style.top = getValueInRange(top, 0, maxTop) + 'px';
  
  // 设置x、y坐标在当前规定范围内
  x = getValueInRange(e.clientX, offsetX, offsetX + boxWidth);
  y = getValueInRange(e.clientY, offsetY, offsetY + boxHeight);
}

// 在全局松开鼠标即设置draggable为false不拖动
document.onmouseup = (e) => {
  draggable = false;
}
```