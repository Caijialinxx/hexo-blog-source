---
title: 一些较重要的HTML标签的学习笔记
date: 2018-02-13 15:32:04
tags: [HTML, Notes]
---
<small>这篇博客将会记录我对一些HTML标签的学习，在本文提到的知识点基本是常用的，如果需要了解详细的标签及其属性用法，可以查询 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element) 。</small>



## `<iframe>` 标签
> Usage：将另一个页面嵌入到当前页面中。详情请[点击](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/iframe)。

### 代码示例
```html
<iframe src="https://caijialinxx.github.io/archives/" name="xxx" frameborder="0" width="100%" height="400px"></iframe>
```

### 结果预览
<pre>
<iframe src="https://caijialinxx.github.io/archives/" name="xxx" frameborder="0" width="100%" height="400px"></iframe>
</pre>

### 重点总结
1. `src` 属性可接网址或相对地址
2. `name` 属性与 `<a>` 标签的 `target` 属性结合使用，可以使 `<a>` 标签的链接在 iframe 容器中呈现
3.  一般情况下都添加 `frameborder="0"` 的属性，使其更美观
4. `width` 属性和 `height` 属性可以设置容器的宽高


## <span id="a">`<a>` 标签</span>
> Usage：创建一个到其他网页、文件、同一页面内的位置、电子邮件地址或任何其他URL的超链接。详情请[点击](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a)。

### 代码示例
```html
<a href="//music.163.com/playlist/1998046799/1300657788?userid=1300657788" target="xxx">分享我的歌单，让你轻松下</a><br>
<a href="#a">点这里将会跳到标题 `<a>标签` </a><br>
<a href="javascript: alert('Hello, I am Here!');">点这里将会执行一个 JavaScript 脚本</a><br>
```

### 结果预览
<pre style="white-space: normal">
<a href="//music.163.com/playlist/1998046799/1300657788?userid=1300657788" target="xxx">分享我的歌单，让你轻松下（翻上去iframe里听哦）</a><br>
<a href="#a">点这里将会跳到本章节 &lt;a&gt;标签 处</a><br>
<a href="javascript: alert('Hello, I am Here!');">点这里将会执行一个 JavaScript 脚本</a><br>
</pre>

### 重点总结
1. `href` 属性的取值：
    - 浏览器支持的协议地址，如 `http://...` 、 `ftp://...` 等（可以使用无协议绝对地址，自动继承当前页面的协议）
    - 相对地址，如 `#xxx`（定位到页面特定锚点，如果是 `#top` 或者 `#` 则直接跳到页面顶部）、 `?yyy=xxx` 、 `./xxx`（当前网页所在 URL 下的文件或文件夹）
    - 伪协议，如 `javascript: alert(1);` 、 `javascript:;`（这可以实现点击&lt;a&gt;标签没有动作，满足一些特殊需求）
2. `target` 属性的取值：
    - `<iframe>` 的 `name` 属性值 —— 这将会使 `<a>` 标签的 URL 在 `<iframe>` 中呈现
    - `_blank`  —— 在新的标签页打开
    - `_self`  —— 在当前页面跳转
    - `_parent`  —— 在父框架或浏览上下文加载，若无父框架或浏览器上下文，则此选项等同 `_self` 。
    - `_top`    在顶层浏览上下文加载。若没有父框架或浏览上下文，则此选项等同 `_self` 。
3. `download` 属性：下载文件
4. 发送 GET 请求（<kbd>F12</kbd> 打开开发者工具，查看 Network ，即可发现当点击一个 `<a>` 链接时，请求方式为 GET ）
    


## `<form>` 标签
> Usage：表示了文档中的一个区域，这个区域包含有交互控制元件，用来向 web 服务器提交信息（ GET 请求或 POST 请求）。详情请[点击](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/form)。

### 代码示例
```html
<form action="" method="get">
  <!-- 表单中必须要有提交按钮，才能发送请求 -->
  <input type="submit">
</form>
```

### 重点总结
1. `method` 属性可指定为 `GET` 或 `POST`
2. `action` 属性指定跳转路径，可以接相对路径（如./anthorpage.html）
3. POST 请求发送后，检查 Headers 有第四部分 Form-Data ，其中第三部分 Request Headers 指定第四部分为 application/x-www-form-urlencoded
4. `target` 属性同 `<a>` 标签
5. 提交表单后会自动刷新页面



## `<input>` 标签
> Usage：用于创建交互式控件，以便接受来自用户的数据。详情请[点击](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input)。

### 代码示例
```html
账号：<input type="text" name="userid"><br>
密码：<input type="password" name="pwd"><br>
性别：<input type="radio" name="sex" value="female">女 <input type="radio" name="sex" value="male" selected>男<br>
爱好：<input type="checkbox" name="hobby" value="swimming">游泳<input type="checkbox" name="hobby" value="running">跑步<input type="checkbox" name="hobby" value="riding">骑行
```

### 结果预览
<pre>
账号：<input type="text" name="userid"><br>
密码：<input type="password" name="pwd"><br>
性别：<input type="radio" name="sex" value="female">女 <input type="radio" name="sex" value="male" selected>男<br>
爱好：<input type="checkbox" name="hobby" value="swimming">游泳 <input type="checkbox" name="hobby" value="running">跑步 <input type="checkbox" name="hobby" value="riding">骑行
</pre>

### 重点总结
1. `type` 属性的取值
    - `submit` —— 提交按钮
    - `button` —— 普通按钮
        PS：`<button>` 标签如果没有 `type` 值且 form 表单里只此一个按钮，那么 `<button>` 标签会自动升级为提交按钮；如果有 `type="button"` ，则是一个普通按钮。另外 `<button>` 标签可以有子标签而 `<input type="button">` 标签不能。
    - `checkbox` —— 复选框
        若同一选项的 `name` 值相同，那么数据传输时将会被集合成一个数组。
    - `radio` —— 单选框
        在该类型下 `name` 属性值必须一样，使得 value 唯一。
    - `password` —— 密码框
    - ……
2. `id` 属性
    可唯一指定一个元素，此外还可以与 `<label>` 标签结合使用。其 `for` 属性值指向 `id` 属性值，或直接包裹在 `<input>` 标签外，即可使用户点击其文本即可选中 input 焦点，例如：
    ```html
    <input type="checkbox" id="xixi"><label for="xixi">嘻嘻</label>
    <label><input type="checkbox" id="haha">哈哈</label>    
    ```
    <pre style='white-space: normal'>
    <input type="checkbox" id="xixi"><label for="xixi">嘻嘻</label>
    <label><input type="checkbox" id="haha">哈哈</label> 
    </pre>



## `<select>` 标签
> Usage：用于创建选项菜单。详情请[点击](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/select)。

### 代码示例
```html
<select name="value">
  <option value="" disabled>-</option>
  <option value="1" selected>1</option>
  <option value="2">2</option>
</select>
```

### 结果预览
<pre>
<select name="value">
  <option value="" disabled>-</option>
  <option value="1" selected>1</option>
  <option value="2">2</option>
</select>
</pre>

### 重点总结
1. `multiple` 属性 —— 多选，按下 <kbd>Crtl</kbd> 键即可多选。
2. `required` 属性 —— 规定select的值不能为空。
3. 必须有 `<option>` 子标签， `<option>` 子标签中的属性有：
    - `disabled` 表示不可选
    - `selected` 表示默认选中



## `<textarea>` 标签
> Usage：可以进行多行纯文本编辑。详情请[点击](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/textarea)。

### 代码示例
```html
<textarea name="textarea" rows="6" cols="30">Write something here...</textarea>
```

### 结果预览
<pre>
<textarea name="textarea" rows="6" cols="30" placeholder="Write something here..."></textarea>
</pre>

### 重点总结
1. 默认可调整，若添加 `style="resize:none"` 属性，则固定宽高
2. `cols` 属性 —— 指定宽度
3. `rows` 属性 —— 指定高度（若CSS行高变化则可能会不同）
4. `readonly` 属性 —— 不允许用户修改元素内文本，能让用户点击和选择元素内的文本。如果在表单里，这个元素的值依旧会跟随表单一起提交。



## `<table>` 标签
> Usage：表示表格数据，即通过二维数据表表示的信息。详情请[点击](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/table)。

### 代码示例
```html
<table border="1">
    <colgroup>
        <col width="60px"><col bgcolor="red" width="80px"><col width="60px">
    </colgroup>
    <thead>
        <tr><th>NO.</th><th>Name</th><th>Age</th></tr>
    </thead>
    <tbody>
        <tr><th>1</th><td>Caaa</td><td>20</td></tr>
        <tr><th>2</th><td>Jaff</td><td>21</td></tr>
    </tbody>
    <tfoot>
        <tr><th>AvgAge</th><td></td><td>20.5</td></tr>
    </tfoot>
</table>
```

### 结果预览
<pre>
<table border="1">
    <colgroup>
        <col width="60px"><col bgcolor="red" width="80px"><col width="60px">
    </colgroup>
    <thead>
        <tr><th>NO.</th><th>Name</th><th>Age</th></tr>
    </thead>
    <tbody>
        <tr><th>1</th><td>Caaa</td><td>20</td></tr>
        <tr><th>2</th><td>Jaff</td><td>21</td></tr>
    </tbody>
    <tfoot>
        <tr><th>AvgAge</th><td></td><td>20.5</td></tr>
    </tfoot>
</table>
</pre>

### 重点总结
1. 子元素有 `<thead>` 、`<tbody>` 、`<tfoot>` 、`<colgroup>` 。 `<tr>` 、`<td>` 不是它的子元素。如果省略不写子元素，不会报错，只会自动更正。
2. 注意，上文中的决定了样式的属性已不建议使用或者已废弃，若需要改变样式请直接使用 CSS 。