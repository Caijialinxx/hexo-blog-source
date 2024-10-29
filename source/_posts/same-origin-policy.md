---
title: 浏览器的同源策略及规避方法
date: 2018-06-01 18:15:05
tags: [AJAX, JavaScript]
---

## 同源策略
> In computing, the same-origin policy is an important concept in the web application security model. Under the policy, a web browser permits scripts contained in a first web page to access data in a second web page, but only if both web pages have the same origin. 

这是来自[维基百科](https://en.wikipedia.org/wiki/Same-origin_policy)的解释。大意就是两个网页在相同的来源的前提下，允许A网页包含的脚本访问B网页的数据。这保证了用户的信息安全，防止某个网站的恶意脚本通过操作 DOM 来获取到敏感数据。

### 同源的条件
所谓的“同源”，需要满足三个条件：
1. 协议相同
2. 域名相同
3. 端口相同

只有同时满足了这三个条件，才能是同源。拿 `http://www.example.com/dir/page.html` 来举个例子，它的协议是 `http://` ，域名是 `www.example.com` ，端口号是默认的 `80` ，那么以下例子中
```
http://www.example.com/dir2/other.html  （同源）
http://username:password@www.example.com/dir2/other.html  （同源）
https://www.example.com/dir/other.html  （不同源，协议不同）
http://example.com/dir/other.html   （不同源，域名不同）
http://www.example.com:81/dir/other.html  （不同源，端口号不同）
```
有同学可能会说 `https://baidu.com` 和 `https://www.baidu.com` 访问结果相同啊，为什么是会域名不同呢？注意，域名相同是指完全相同而不是相似，`baidu.com` 是一级域名（此说法有误，但绝大部分人都这么叫，《计算机网络》中将其解释为二级域名），而 `www.baidu.com` 为二级域名（若前一个是二级域名那么这个对应的就是三级域名），当然是不同的两个域名。访问结果相同是因为百度公司设置的 DNS 解析的关系，[这篇文章](https://blog.csdn.net/meimingming/article/details/9038223)或许能够解答你，但不能因为结果相同就说它们是一样的。不信可以去访问一下 `http://www.12306.cn` 和 `http://12306.cn` ，就能让你“眼见为实”了。

### 带来的问题
1. 一级域名相同，只是二级域名不同的同一所有者的网页被限制（Cookie、LocalStorage、IndexDB的读取）
2. 无法跨域发送 AJAX 请求
3. 无法操作 DOM

- Q：为什么 Form 表单可以跨域发送请求，而 AJAX 不可以。
  
  A：因为 Form 表单提交之后会刷新页面，所以即使跨域了也无法获取到数据，所以浏览器认为这个是安全的。而 AJAX 最大的优点就是在不重新加载整个页面的情况下，更新部分网页内容。如果让它跨域，则可以读取到目标 URL 的私密信息，这将会变得非常危险，所以浏览器是不允许 AJAX 跨域发送请求的。



## 规避方法
1. [`document.domain`属性](#document.domain)
2. [CORS](#CORS)
3. [JSONP](#JSONP)
4. [Cross-document messaging](#Cross-document_messaging)
5. [WebSocket](#WebSocket)

### document.domain
假设你已登录并正浏览腾讯网 `http://www.qq.com/` ，此时你突然想看腾讯视频，于是你点击腾讯网提供的快捷链接“视频”，浏览器加载 `https://v.qq.com/` 。你想查看自己以往的观看记录，因为这两个网页的二级域名不同，`v.qq.com` 无法获取到你在 `www.qq.com` 的登录信息，所以无法查看观看记录，于是你不得不重新登录一次。
这一趟操作下来，是不是很不人性化？如果能避免这个同源限制，直接读取 Cookie 自动登录就好了。腾讯当然也是这么想的。

> If two windows (or frames) contain scripts that set domain to the same value, the same-origin policy is relaxed for these two windows, and each window can interact with the other. -- [document.domain property, Wiki](https://en.wikipedia.org/wiki/Same-origin_policy#document.domain_property)

从上面的例子中，我们只要将这两个网页的设置相同的域名，即可共享 Cookie 。即
```
document.domain = 'qq.com'
```



### CORS
> This standard extends HTTP with a new Origin request header and a new Access-Control-Allow-Origin response header. -- [Cross-Origin Resource Sharing, Wiki](https://en.wikipedia.org/wiki/Same-origin_policy#Cross-Origin_Resource_Sharing) 

跨源资源共享（Cross-Origin Resource Sharing，简称 CORS ），它是 W3C 标准，通过一个新的 Origin 请求头和一个新的 Access-Control-Allow-Origin 响应头来扩展 HTTP ，即可解决 AJAX 跨源请求的问题。
假设 `http://www.qq.com` 要向 `https://v.qq.com` 发送 AJAX 请求，那么他们需要这样设置：
```javascript
/* http://www.qq.com 前端发起 AJAX 请求 */
let req = new XMLHttpRequest()
req.onreadystatechange = () => {
  if(req.readyState === 4) {
    if(req.status >=200 && req.statue < 400) {
      console.log('success')
    } else {
      console.log('failed')
    }
  }
}
req.open('GET', 'https://v.qq.com')
req.send()
```
```javascript
/* https://v.qq.com 被请求的后台 */
...
响应.setHeader('Access-Control-Allow-Origin', 'http://www.qq.com')
...
```
那么我们在 `http://www.qq.com` 下中打开 Chrome 的开发者工具的 Network ，即可看到 `https://v.qq.com` 的请求状态码为 `200` ，同时请求头中添加了：
```
Origin: http://www.qq.com
```
以及响应头中添加了：
```
Access-Control-Allow-Origin: http://www.qq.com
```


### JSONP
JSONP 是一种使用模式，它通过动态创建 `<script>` 标签来向跨源网址发送请求，其中这个请求的查询字符串中含有 callback 参数，用来指定回调函数的名字。服务器收到请求后，将数据放在一个指定名字的回调函数里传回来。示例代码如下：
```javascript
/* http://example1.com 发送请求 */
let script = document.createElement('script')
let functionName = `c${parseInt(Math.random() * 10000, 10)}`

window[functionName] = (res) => {
  alert(res)
}

script.src = `http://example2.com/?callback=${functionName}`
script.onload = (e) => {
  e.target.remove()
  delete window[functionName]
}
script.onerror = (e) => {
  e.target.remove()
  delete window[functionName]
}

document.body.appendChild(script)
```
```javascript
/* http://example2.com 发送响应 */
...
let callbackName = 请求方URL.callback
响应.statusCode = 200
响应.setHeader('Content-Type', 'application/javascript')
响应.write(`${callbackName}.call(undefined, "Success")`)
响应.end()
...
```
请求方 `http://example1.com` 动态创建 `<script>` 之后，其 `src` 属性指向被请求方的路径 `http://example2.com` 并在尾部加上 `callback` 参数，如 `?callback=xxx` 。被请求方获取到请求方的 `callback` 参数，并发回响应构造函数调用，如 `xxx.call(undefined, 'yyy')` 。那么请求方将会弹窗显示“yyy”。



### Cross-document messaging
> Cross-document messaging allows a script from one page to pass textual messages to a script on another page regardless of the script origins... A script in one page still cannot directly access methods or variables in the other page, but they can communicate safely through this message-passing technique. -- [Cross-document messaging, Wiki](https://en.wikipedia.org/wiki/Same-origin_policy#Cross-document_messaging)

跨文档通信（Cross-document messaging），维基百科解释得很清楚了，它允许一个网页的脚本向另一个网页传递文本消息，尽管他们不同源。不过它们也只能通过此技术安全地传递文本消息而已，仍无法直接访问另一个页面的方法和变量。
这是 HTML5 引入的新 API ，它为 window 对象新增了 `window.postMessage()` 方法。它允许用户跨窗口通信，不管是否同源。

例子如下：
我在父窗口 JS Bin 中打开一个新窗口，子窗口的 URL 指向百度 `https://www.baidu.com` ，并且在百度页面的控制台中定义了 `window.onmessage` 事件，使其输出 `event` 对象。
![在jsbin的代码.png](https://i.loli.net/2018/06/01/5b10fb4408d03.png)
```html
<button id='btn'>postMessage</button>
<script src="//code.jquery.com/jquery-2.1.1.min.js"></script>
```
```javascript
// 父窗口 http://js.jirengu.com/wirixavoxo/1/edit?js,output
var x = window.open('https://www.baidu.com', 'baidu');
$(btn).click(function() {
  x.postMessage('from father', 'https://www.baidu.com');
});

window.addEventListener('message', function(e) {
  console.log(e);
}, false);
```
```javascript
// 子窗口 https://www.baidu.com/
window.addEventListener('message', function(e) {
	console.log(e)
}, false)
```

当我点击父窗口中的按钮时，触发其 `click` 事件，向子窗口发送了一条消息，在子窗口监听 `message` 事件的结果如下图所示：
![在百度中监听的message事件结果.png](https://i.loli.net/2018/06/01/5b10fb441113a.png)
我们可以看到 `e.data` 就是父窗口中发送的消息 `"from father"` ， `e.origin` 为父窗口的 URI `"http://js.jirengu.com"` ，且 `e.type` 为 `"message"` 。

然后我在子窗口中分别声明了 `String` 变量、 `Function` 变量和 `Object` 变量，其中 `Object` 变量是获取了百度页面的搜索按钮。根据结果可以发现，在子窗口定义的 `Function` 类型和 `HTMLElement` 类型（即使这个HTML元素没有被插入到页面中）不被允许发送，根据控制台的报错提示，是因为它们“不能被克隆”。
![在百度中尝试给jsbin发送消息.png](https://i.loli.net/2018/06/01/5b10fb4417cba.png)
```javascript
// 子窗口 https://www.baidu.com/
var str = 'this is a string!'
window.opener.postMessage(str, 'http://js.jirengu.com/wirixavoxo/1/edit?js,output')

var func = function() {
  console.log('this is a function')
}
window.opener.postMessage(func, 'http://js.jirengu.com/wirixavoxo/1/edit?js,output')

var btn = document.getElementById('su')
window.opener.postMessage(btn, 'http://js.jirengu.com/wirixavoxo/1/edit?js,output')
```
相应地，在父窗口中我们可以看到结果也是只显示了 `str` 这个变量的输出结果。`e.origin` 是子窗口 `"https://www.baidu.com"` 。
![在jsbin中监听的message事件结果.png](https://i.loli.net/2018/06/01/5b10fb4417be9.png)
若想知道其他类型的输出结果，请自行运行测试哦。有些结果我没有放出来，但结论已经在上面了。再阐述一遍就是：
1. `Function` 类型无法传输
2. `HTMLElement` 类型，无论是获取页面的元素，还是自行新创建但还没被插入到页面的，也无法传输
3. 只要是能传输的类型，无论你是 `String` 、 `Number` 、 `undefined` 、 `null` 、 `Boolean` 或 `Object` 等类型，传到父窗口后查看 `typeof e.data` ，结果都是 `string` 。至于为什么，我也不清楚，我猜想是 `message` 事件中对那些数据类型进行了强制转换，或者是因为这个功能还在开发中，所以这个 API 并不成熟。



### WebSocket
> Modern browsers will permit a script to connect to a WebSocket address without applying the same-origin policy. However, they recognize when a WebSocket URI is used, and insert an Origin: header into the request that indicates the origin of the script requesting the connection. To ensure cross-site security, the WebSocket server must compare the header data against a whitelist of origins permitted to receive a reply. -- [WebSockets, Wiki](https://en.wikipedia.org/wiki/Same-origin_policy#WebSockets)

下面这个例子来自[维基百科](https://en.wikipedia.org/wiki/WebSocket#Protocol_handshake)，客户端的请求报头如下：
```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com

```
服务器收到这个请求之后，查询 `Origin` 字段的 URL 是否在其白名单内，如果是就返回以下报文：
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat

```
一旦连接建立成功，客户端和服务器就可以以全双工模式来回传输数据。



## AJAX 的规避方法
AJAX 适用的规避方法有 CORS（跨源资源共享）、 JSONP 及 WebSocket。

### CORS 与 JSONP 的区别
1. CORS 比 JSONP 要更强大一点，因为 JSONP 只支持的 GET 请求，而 CORS 支持所有 HTTP 请求。
2. 但对于老式浏览器的支持上， JSONP 更占优势，且能向不支持 CORS 的网站发送请求。



## 参考资料
1. [AJAX](http://javascript.ruanyifeng.com/bom/ajax.html)
2. [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
3. [跨域多个域名白名单](http://www.jb51.net/article/109725.htm)
4. [DNS解析过程详解](https://blog.csdn.net/meimingming/article/details/9038223)
5. [HTML5 postMessage 和 onmessage API 详细应用](https://www.cnblogs.com/sugar-tomato/p/4497108.html)