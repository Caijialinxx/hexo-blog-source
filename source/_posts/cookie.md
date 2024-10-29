---
title: HTTP 牌的 Cookie
date: 2018-07-12 15:38:29
tags: [Notes, HTTP]
---

## Cookie 是什么？
今天要介绍的 Cookie ，不是皇冠丹麦牌的哦，而是我们 HTTP 牌的~
> HTTP Cookie（也叫Web Cookie或浏览器Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。

由上面的介绍，提炼成简短的一句话就是——**Cookie是一段数据**。 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies) 的介绍已非常清晰易懂。目前使用最广泛的是由网景公司在其制定的标准上进行扩展后的产物。


## 能吃吗？
HTTP Cookie 当然不能吃，但是！它可以用于**识别用户身份**和**记录用户操作历史**。
- 识别用户身份
  例如，当用户登录一个网站时，如果用户需要记住登录，那么服务器就会发送一个包含此用户登录凭证（用户名和密码的某种加密形式）的 Cookie 到用户本地（将会被存储在计算机的硬盘中），那么在 Cookie 的有效期内再次访问这个网站，浏览器就会自动带上这个 Cookie 去访问服务器，服务器识别到这个 Cookie ，就可以免去用户登录这个操作。
- 记录操作历史
  假设现有一个用户在网上购物，由于 HTTP 协议是无状态的，即服务器不知道用户上一次做了什么，如果不依靠其他的手段，服务器是无法记录到用户买了什么的，而 Cookie 就可以弥补这一缺陷——用户每选购一个商品，服务器就可以向用户发送一段 Cookie ，记录那个商品的信息。这样每当用户访问新的商品页面，浏览器就会把 Cookie 发给服务器，服务器在这个 Cookie 里追加信息。最后，结账的时候，服务器读取接受到的 Cookie 就可以知道用户一共选购了什么商品。

> MDN 中详细描述了 Cookie 的使用场景：
> - 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
> - 个性化设置（如用户自定义设置、主题等）
> - 浏览器行为跟踪（如跟踪分析用户行为等）


## 有什么口味？
没有榛仁味，也没有果干味。按它的储存位置，可以分为**内存Cookie**和**硬盘Cookie**：
- 内存 Cookie 由浏览器维护，保存在内存中，浏览器关闭后就消失了，其存在时间是短暂的。
- 硬盘 Cookie 保存在硬盘里，有一个过期时间，除非用户手动清理或到了过期时间，硬盘Cookie不会被删除，其存在时间是长期的。

所以，按存在时间，还可分为**会话期Cookie**和**持久性Cookie**：
- 会话期 Cookie 不需要指定过期时间（ `Expires` ）或者有效期（ `Max-Age` ）。需要注意的是，有些浏览器提供了会话恢复功能，这种情况下即使关闭了浏览器，会话期 Cookie 也会被保留下来，就好像浏览器从来没有关闭一样。
- 持久性 Cookie 可以指定一个特定的过期时间（ `Expires` ）或有效期（ `Max-Age` ）。


## 制作的关键工序
- 后端人员在响应头中设置 Cookie
    Node.js 下的后端代码如下：
    ```js
    response.setHeader("Set-Cookie", "user=caijialinxx@foxmail.com");
    ```
    如果要设置多个 Cookies ，那么使用数组包含每一个元素即可，即：
    ```js
    response.setHeader("Set-Cookie", ["user=caijialinxx@foxmail.com", "language=javascript"]);
    ```
- 用户在浏览器中修改 Cookie
    一些浏览器自带或安装开发者工具允许用户查看、修改或删除特定网站的 Cookies 信息。这也带来了一个问题，用户可以伪造 Cookie 来操作网页。


## 保质期多久？
后端开发人员可以加多点防腐剂，让 Cookie 的有效期长一点，也可以让它几分钟之后就过期，取决于后端人员如何设置。一般默认的有效期是20分钟，每个浏览器的默认时长可能不同。

这个防腐剂就是上面提到的 `Expires` 或者 `Max-Age` 属性。
- `Expires` 属性
    它可以指定一个具体的到期时间，到了指定时间之后浏览器就不会再保存这个 Cookie 。例如：
    ```js
    response.setHeader("Set-Cookie", "user=caijialinxx@foxmail.com; Expires=Thu, 12 Jul 2018 02:54:00 GMT")
    ```
    那么，可以看到收到的响应头中含有
    ```
    Set-Cookie: user=caijialinxx@foxmail.com; Expires=Thu, 12 Jul 2018 04:00:00 GMT
    ```
    ![响应头中的Set-Cookie字段](https://i.loli.net/2018/07/12/5b46c913a2451.png)
    且新增了这条 Cookie 如下图所示：
    ![设置了Expires的Cookie](https://i.loli.net/2018/07/12/5b46c913b9072.png)
    但是由于这个时间是依赖本地时间来决定 Cookie 具体何时过期，所以没有办法保证 Cookie 一定会在服务器指定的时间过期。
- `Max-Age` 属性
    它可以规定从现在开始，在指定的时间后 Cookie 到期，单位为秒。例如：
    ```js
    response.setHeader("Set-Cookie", "user=caijialinxx@foxmail.com; Max-Age=60")
    ```
    那么，可以看到收到的响应头中含有
    ```
    Set-Cookie: user=caijialinxx@foxmail.com; Max-Age=60
    ```
    ![响应头中的Set-Cookie字段](https://i.loli.net/2018/07/12/5b46ca8c202a2.png)
    我们可以看到 Cookie 在 `2018-07-12T03:30:24.539Z` 后过期，这个时间是根据我们启动服务器后，增加60s后得到的时间。
    ![a1d3d7198a21adbb2c8248a80f20590.png](https://i.loli.net/2018/07/12/5b46ca8c1648d.png)


## 食品安全认证
在第4节中我们知道了，用户可以借助浏览器的开发者工具修改 Cookie ，那么这意味着我们可以篡改网站的 Cookie 如下图所示：
![在开发者工具中读写Cookie](https://i.loli.net/2018/07/12/5b46e92152a80.png)
具体的危害可以查看[偷窃Cookies和脚本攻击——维基百科](https://zh.wikipedia.org/wiki/Cookie#%E5%81%B7%E7%AA%83Cookies%E5%92%8C%E8%84%9A%E6%9C%AC%E6%94%BB%E5%87%BB)。
为了避免这种情况，我们可以添加 `HttpOnly` 标记：
- `HttpOnly` 可以使 Cookie 无法通过 JavaScript 的 `document.cookie` 读写。
    ```js
    response.setHeader("Set-Cookie", "user=caijialinxx@foxmail.com; HttpOnly")
    ```
    响应头中含有
    ```
    Set-Cookie: user=caijialinxx@foxmail.com; HttpOnly
    ```
    但可以在 Application 中更改，如下图所示。
    ![标记了HTTPOnly后Cookie仍可被更改](https://i.loli.net/2018/07/12/5b46f6f3dea55.png)

此外，还有 `Secure` 标记：
- `Secure` 可以使 Cookie 只能通过被 HTTPS 协议加密过的请求发送给服务器，如果不是 HTTPS 协议，那么浏览器将不会接受这个 Cookie 。
    ```js
    response.setHeader("Set-Cookie", "user=caijialinxx@foxmail.com; Secure")
    ```
    响应头中含有
    ```
    Set-Cookie: user=caijialinxx@foxmail.com; Secure
    ```
    ![响应头返回的Set-Cookie中含有Secure标记](https://i.loli.net/2018/07/12/5b46fbd56617b.png)
    因为此网站是 `http://localhost:8888` ，所以在请求头中浏览器无法发送 Cookie 。
    ![请求头中无Cookie字段](https://i.loli.net/2018/07/12/5b46fb656f489.png)
    但即便设置了 `Secure` 标记，敏感信息也不应该通过 Cookie 传输，因为Cookie有其固有的不安全性，`Secure` 标记也无法提供确实的安全保障。从 Chrome 52 和 Firefox 52 开始，不安全的站点（http:）无法使用 Cookie 的 `Secure` 标记。


## 它的不美味之处
- Cookie 会被附加在每个 HTTP 请求中，所以无形中增加了流量，带来额外的开销。
- 由于在 HTTP 请求中的 Cookie 是明文传递的，所以安全性成问题，除非用 HTTPS 。
- Cookie 的大小一般限制在 4KB 左右，对于复杂的存储需求来说是不够用的。
- 识别不精确， Cookie 只能定位到用户、浏览器和计算机这三者的组合，一旦有一个不同， Cookie 的内容就会不同或者是存储位置不一样。
- 不准确的状态——浏览器的“回退”按键会使 Cookie 无法记录上一次的用户操作。
- Cookies 在某种程度上说已经严重危及用户的隐私和安全，它会被利用于投放广告。


## 总结
1. Cookie 是一段数据，它由服务器发送给浏览器并保存到本地，存储着用户的一些信息（如用户登录状态、个性化设置等），在此浏览器下一次访问此服务器时，这段 Cookie 会被附加在 HTTP 请求中发送给服务器。
2. Cookie 的大小一般为 4KB ，默认有效期一般为 20mins ，但可以通过 `Expires` （UTC格式）或 `Max-Age` （秒数）来设置其过期时间。
3. Node.js 设置 Cookie 的 API ： `response.setHeader("Set-Cookie", "name=value")` 。
4. 浏览器读写 Cookie 的 API ： `document.cookie` 。若要阻止用户通过 JavaScript 来读写 Cookie ，只需要添加 `HttpOnly` 标记。
5. `Secure` 标记可以使 Cookie 只能通过 HTTPS 加密过的请求发送服务器，如果不是 HTTPS 请求，浏览器会自动忽略。
6. 由于 Cookie 会被附加在 HTTP 请求中，且是明文传递，所以会增加流量以及有安全问题。建议使用 Web storage API （ `sessionStorage` 或 `localStorage ` ）或 IndexedDB 。


## 扩展阅读
在这篇笔记写到一半的时候，我看到了 MDN 的介绍，我觉得它写得已经很好了，一度有不想写直接抛出链接的想法........
然鹅，这篇笔记可是我自己消化的东西啊。不管怎么样，都还是应该要自己写出来。只不过我还是会把 MDN 的链接放在下面作为扩展阅读，当然还有其他一些优秀的博客笔记。
- [HTTP Cookie, MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)
- [说说HttpOnly Cookie的作用](http://ulricqin.com/post/httponly-cookie/)
- [Cookie, JavaScript 标准参考教程](http://javascript.ruanyifeng.com/bom/cookie.html)——阮一峰
- [What is httponly cookie?](https://latesthackingnews.com/2017/07/03/what-is-httponly-cookie/) ——外文阅读，有兴趣请戳