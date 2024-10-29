---
title: HTTP 的请求和响应
date: 2018-09-09 22:02:04
tags: [Notes, HTTP]
---

HTTP 协议是用来指导服务器和浏览器之间如何进行沟通的，一般来说端口默认是 `80` 。浏览器通过向服务器发送 HTTP 请求，来请求服务器中的资源。服务器根据这个请求，返回相应的响应。最后浏览器下载这个响应的实体内容。

## HTTP请求
根据 [RFC 2616 的第五章 Request](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) ，一个请求基本包括请求行、请求头、 CRLF 至少这三部分内容，有时还有第四部分请求实体。
![RFC 2616规范中HTTP请求的结构](https://i.loli.net/2018/10/09/5bbc667c2232a.png)

例如，我们使用下列命令向 `https://www.baidu.com` 发送 POST 请求（[点击查看命令的解释](https://explainshell.com/explain?cmd=curl+-X+POST+-d+%27%7B%22name%22%3A%22caijialinxx%22%7D%27+-H+%27Content-Type%3A+application%2Fjson%27+-s+-v+https%3A%2F%2Fwww.baidu.com)），可以看到它的请求报文如下图所示：
```
curl -X POST -d '{"name":"caijialinxx"}' -H 'Content-Type: application/json' -s -v https://www.baidu.com
```
![向 https://www.baidu.com 发送POST请求的请求报文](https://i.loli.net/2018/10/09/5bbc72d1e70f7.png)

- 第一部分即为请求行，分别包括 `请求方法` 、 `请求URI` 、 `HTTP版本` ，它们之间使用英文空格 `SP` 分隔，此外，最后是一个换行符 `CRLF` 。
- 第二部分是以 `key: value` 形式组成的请求头。其中包括 request-header （如 `Host` `User-Agent` `Accept`）和 entity-header （如 `Content-Type` `Content-Length`）。每一对 `key: value` 请求头后也都有一个 `CRLF` 。
- 第三部分是固定不变的单独一行 `CRLF` ，用于分隔第二部分和第四部分。
- 第四部分是发送请求时带上的请求实体，即 `{"name":"caijialinxx"}` 。命令行在第二部分中添加的 `Content-Type: application/json` 请求头，标示着第四部分是 JSON 格式的数据。

我们发送请求的方法有以下几种：
- **GET**
  GET 方法是 HTTP 请求中最常用的方法，通常用于请求服务器中的某个资源。此方法应只用来获取数据。
- HEAD
  HEAD 方法与 GET 方法类似，但是服务器在响应中只返回头部，而不返回响应实体。
- **POST**
  此方法最初是用来向服务器输入数据的。实际上经常与 HTML 表单结合使用，表单被提交时，数据通常被附加到请求实体中发送给服务器。
- **PUT**
  此方法让服务器用请求的实体部分来创建一个由请求中的 URL 命名的新文档，如果这个 URL 已存在，那么此次请求的主体会替代原有的主体。
- **PATCH**
  用于请求服务器修改指定资源的某个部分。
- **DELETE**
  向服务器请求删除指定 URL 对应的资源。
- OPTIONS
  请求 Web 服务器告知其支持的各种功能，如支持的方法或对请求某些资源支持的方法。
- CONNECT
  建立一个到由目标资源标识的服务器的隧道。
- TRACE
  沿着到目标资源的路径执行一个消息环回测试。



## HTTP响应
根据 [RFC 2616 的第六章 Response](https://www.w3.org/Protocols/rfc2616/rfc2616-sec6.html#sec6) ，一个响应包括状态行、响应头、 CRLF 和响应实体。
![RFC 2616规范中HTTP响应的结构](https://i.loli.net/2018/10/09/5bbc81ec9067c.png)

在上一部分中（[HTTP 请求](#HTTP请求)），我们向 `https://www.baidu.com` 发送了一个 POST 请求，它紧接着返回了响应给我们，让我们来看看它的响应报文：
![服务器返回的响应报文](https://i.loli.net/2018/10/09/5bbc84836492c.png)

- 第一部分即为状态行，分别包括 `HTTP版本` 、 `状态码` 、 `状态解释` ，它们之间使用英文空格 `SP` 分隔，此外，最后是一个换行符 `CRLF` 。在此报文中我们可以看到状态码是 `302` ，表示页面暂时被另一个 URI 所替代，但客户端在未来的请求中仍应使用当前请求的 URI 。
- 第二部分是以 `key: value` 形式组成的响应头。其中包括 general-header (`Connection` , `Date`) 、 response-header (`ETag` , `Server`) 和 entity-header (`Content-Type` , `Content-Length`) 。每一对 `key: value` 请求头后也都有一个 `CRLF` 。
- 第三部分是固定不变的单独一行 `CRLF` ，同请求报文的第三部分。
- 第四部分是响应实体。在此报文第二部分中我们可以看到 `Content-Type: text/html` 响应头，表明第四部分是一个 HTML 文档，它有 3824 字节长度的数据。


关于状态码的详解，请参考我的另一篇文章 [一些常见的 HTTP 状态码](https://caijialinxx.github.io/2018/06/27/http-status-code/) 。


## 如何使用开发者工具查看报文
作为一个前端开发人员，学会用开发者工具分析请求是一件很有帮助且不费时间的事情。下面，我将举例如何在 Chrome 中利用开发者工具查看请求报文和响应报文：

1. 按下 <kbd>F12</kbd> 即可打开开发者工具。
2. 点击 Network ，当访问一个网页时，这里将会出现很多请求结果。我以阮一峰老师的个人网站为例，在地址栏中输入 `www.ruanyifeng.com` ，然后可以看到如下请求结果：
![在Chrome中访问www.ruanyifeng.com](https://i.loli.net/2018/10/09/5bbcc122db1e4.png)

3. 我们点开第一个 `www.ruanyifeng.com` ，点击 Request Header 中的 view source，这才是我们该看的格式。于是它的请求报文如下：
![www.ruanyifeng.com的请求报文](https://i.loli.net/2018/10/09/5bbcc11eb4bde.png)
    - 浏览器使用 `GET` 方法向 `http://www.ruanyifeng.com` 发送一个请求，并告诉服务器它接受的响应实体类型、 Cookie 等请求头信息。

4. 然后同样的，点开 Response Header 中的 view source ，看到服务器发回的响应报文如下所示：
![www.ruanyifeng.com的响应报文](https://i.loli.net/2018/10/09/5bbcc11eea954.png)
    - 服务器返回状态码 `302` ，告诉我们这个域名被响应头 `Location` 指向的 URI 所暂时替代，并自动完成了重定向，于是得到了第二个请求结果（箭头指向的红框）。

5. 查看重定向的请求结果：
![home.html的报文](https://i.loli.net/2018/10/09/5bbcc43290425.png)
    - 重定向使得浏览器重新向服务器请求 `http://www.ruanyifeng.com/home.html` 这个资源，在 Response Headers 中我们可以看到此次请求成功（状态码为 `200` ），并且返回的响应实体是一个 HTML 文档，也就是我们看到的这个网页，一共有 4346 字节的内容。这个网页最后一次更新是在格林尼治的 2018-5-24 星期四 05:56:54 等信息。

现在应该知道要如何利用开发者工具查看我们的网络请求了吧~ 好处你用过就知道~ 当然开发者工具还不只有这些功能，在这里我就不做深入研究。

------------------
天色已晚，本文完。

若文中有错误还请指正与包涵！

原文链接：https://caijialinxx.github.io/2018/09/09/http-request-and-response/

转载请注明出处。

参考资料：
- [HTTP 请求方法](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods) —— MDN
- [HTTP 协议之请求](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html#sec5)
- [HTTP 协议之响应](https://www.w3.org/Protocols/rfc2616/rfc2616-sec6.html#sec6)
- [HTTP 协议之状态码](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10)