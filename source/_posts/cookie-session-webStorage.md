---
title: Cookie, Session, LocalStorage & SessionStorage
date: 2018-07-16 17:19:17
tags: [Notes, HTML5, JavaScript]
---

## Cookie
由于刚刚写过一篇介绍 [Cookie](https://caijialinxx.github.io/2018/07/12/cookie/) 的博客，所以我在这里不打算再次介绍多一遍，直接总结如下：
1. Cookie 是一段由服务器发送给浏览器并保存到本地的一段数据，大小一般为 4KB 。
2. 它存储着用户的一些信息，在浏览器再次访问同一个 URL 时会将这段 Cookie 附加到 HTTP 请求中发送给服务器。因此，这会增加流量的消耗。
3. 一般在浏览器关闭（会话结束）时就被删除，但也可以通过 `Expire` 或 `Max-Age` 来设置过期时间。
4. 浏览器可以通过 `document.cookie` 读写 Cookie ，若要阻止此行为，可以在 `Set-Cookie` 头中添加 `HttpOnly` 标记。

详细的还请移步到 [HTTP 牌的 Cookie](https://caijialinxx.github.io/2018/07/12/cookie/) 参阅。


## Session
Session 的实质是存储在服务器里的一小块内存，一般来说是基于 Cookie 实现的。实现过程举例如下：
```js
/* 
 * 1. 当第一次使用 Session 时，服务器要创建一个 SessionID
 *    来作为 Session 中存放用户信息的唯一标识
 */
let session = {}
let sessionID = Math.random().toString().slice(2)       // 假设为 '1234567890'
session[sessionID] = 用户信息       // 此时服务器中已存含有用户信息的 SessionID 对应的内存

/* 2. 服务器通过 Cookie 给用户返回一个 SessionID */
response.setHeader('Set-Cookie', sessionID)
```
```
/* 3. 当用户访问同一个 URL 时，向服务器发送这个内容为其 SessionID 的 Cookie 头 */
Request Header
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 580
Cookie: 1234567890
...

```
```js
/* 
 * 4. 服务器读取 Cookie 中的 SessionID ，
 *    然后去 Session 中读取对应的内存，最终得到用户信息。
 */
if (request.headers.cookie) {
  let sessionID = request.headers.cookie
  let userInfo = session[sessionID]               // 读取用户信息
}
```
值得一提的是， SessionID 是随机生成的。


## LocalStorage
`window.localStorage` 是 HTML5 新增的一个 API ，它遵循同源政策，属于本地存储。它有以下特点：
1. 大小一般为 5MB ，永久有效；
2. 本地存储和读取，不会被附加到 HTTP 请求中。

`window.localStorage` 的使用如下例所示：
```js
// 保存数据到 LocalStorage
localStorage.setItem('key', 'value')           // {"key": "value"}

// 从 LocalStorage 获取数据
let key = localStorage.getItem('key')          // "value"

// 从 LocalStorage 删除保存的数据
localStorage.removeItem('key')                 // {}

// 从 LocalStorage 删除所有保存的数据
sessionStorage.clear();
```


## SessionStorage
`window.sessionStorage` 同样也遵循同源政策，它与 `window.localStorage` 不同的是，它在页面或浏览器关闭（会话结束）时就会被清除。
`window.sessionStorage` 的使用如下例所示：
```js
// 保存数据到 SessionStorage
sessionStorage.setItem('key', 'value')

// 从 SessionStorage 获取数据
var data = sessionStorage.getItem('key')

// 从 SessionStorage 删除保存的数据
sessionStorage.removeItem('key')

// 从 SessionStorage 删除所有保存的数据
sessionStorage.clear()
```


## 总结
四者之间的比较如下：
### Cookie 和 Session
- Cookie 保存在客户端的硬盘里；而 Session 是保存在服务端的内存里。
- Cookie 每次都随着 HTTP 请求发送给服务器；而 Session 只需要通过发送保存在 Cookie 头中的 SessionID 即可从服务器中读取对应内存。
- Cookie 以明文的形式存储，内容可以被用户查看或篡改；而 Session 因为只提供一个随机的 SessionID ，所以无法被用户直接查看。

### Cookie 和 LocalStorage
- Cookie 的大小一般为 4KB ；而 LocalStorage 一般为 5MB 。
- Cookie 一般在浏览器关闭（会话结束）时就失效，后端可以设置 `Expires` 或 `Max-Age` 来改变 Cookie 的过期时间；而 LocalStorage 是永久有效，除非用户手动清除。
- Cookie 会被附加到 HTTP 请求中；而 LocalStorage 不会。

### LocalStorage 和 SessionStorage
- 前者永久有效，除非用户删除；后者在浏览器关闭（会话结束）后就被清空。