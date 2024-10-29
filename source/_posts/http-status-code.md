---
title: 一些常见的 HTTP 状态码
date: 2018-06-27 19:05:02
tags: [Notes, HTTP, 面试]
---

## 1xx Informational responses
- 100 Continue
  服务器已接收到请求报头，客户端应继续发送请求报文。
- 101 Switching Protocols
  服务器已根据客户端的请求切换协议。

## 2xx Success
- **200 OK**
  请求已成功。
- 201 Created
  请求已完成，新的资源已被创建，且其 URI 已随着响应行返回。
- 202 Accepted
  请求已被接受处理，但处理尚未完成。该请求最终可能会也可能不会被执行。该状态码在异步操作的情况下很方便。
- 204 No Content
  服务器已成功处理请求，没有返回任何实体内容。
- 205 Reset Content
  服务器已成功处理请求，且没有返回任何内容。但与 `204` 不同的是，包含此状态码的响应要求请求者重置文档视图。

## 3xx Redirection
- **301 Moved Permanently**
  被请求的资源被永久移动到返回的新的 URI 中。
- 302 Found
  告诉客户端浏览另一个 URL 。 `302` 已被 `303` 和 `307` 所取代。
- 303 See Other
  使用 `GET` 方法在另一个 URI 下找到请求对应的响应。
- **304 Not Modified**
  所请求的资源未修改，服务器返回此状态码时，表示已执行 GET 请求，且由于文件无变化于是不会返回任何资源。
- 305 Use Proxy
  请求的资源只能通过代理服务器获得，响应中提供其代理服务器的地址。
- 307 Temporary Redirect
  请求暂时由另一个 URI 响应，而未来的请求应仍继续使用原始的 URI 。

## 4xx Client errors
- **400 Bad Request**
  服务器无法处理请求，因为客户端的请求有明显的错误（例如请求语法错误、请求过大等）。
- **401 Unauthorized**
  当前请求未通过认证。响应必须包含一个适用于被请求资源的 WWW-Authenticate 信息头来询问用户信息。
- **403 Forbidden**
  请求已被理解但是服务器拒绝执行。与 `401` 不同的是，用户无法通过身份验证来得到请求资源的许可。
- **404 Not Found**
  请求的资源目前无法被找到。
- 405 Method Not Allowed
  请求的方法无法支持请求的资源。例如，一个本应通过 POST 请求来获取数据的表单使用了 GET 请求（a GET request on a form that requires data to be presented via POST），或者是一个只读的资源使用了 PUT 请求（or a PUT request on a read-only resource）。该响应必须返回一个 Allow 头信息用以表示出当前资源能够接受的请求方法的列表。
- 406 Not Acceptable
  请求的资源的内容特性无法满足请求头中的条件，因而无法生成响应实体。
- 410 Gone
  被请求的资源在服务器上已不再可用，并且没有任何已知的转发地址。
- **422 Unprocessable Entity**
  请求的格式是正确的，但是由于语义错误，服务器无法响应。

## 5xx Server errors
- **500 Internal Server Error**
  服务器内部错误，无法完成请求。一般来说，这个问题会在服务器端的源代码错误时出现。
- **502 Bad Gateway**
  服务器充当网关或代理时，收到来自上游服务器的无效响应。
- **503 Service Unavailable**
  服务器当前不可用，原因可能是服务器超载或正在维护。
- 504 Gateway Timeout
  服务器充当网关或代理时，没有及时收到来自上游服务器的响应。


---
本文完。

参考资料：[List of HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) —— 维基百科