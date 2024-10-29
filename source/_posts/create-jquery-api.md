---
title: 使用原生JS实现一个jQuery的API
date: 2018-05-09 16:39:28
tags: [JavaScript, jQuery, Notes]
---

## 原生JS操作
```html
<ul style='border: 1px solid #666'>
  <li id='li'>Hi</li>
</ul>
```

代码结果：
<ul style='border: 1px solid #666'>
  <li id='li'>Hi</li>
</ul>

现在，我想为 `<li>` 标签添加一个样式，
```css
.red { color: red; }
```

并改变文本内容为“Thank you~”。用原生JS的写法就是：
```javascript
li.classList.add('red')
li.textContent = 'Thank you~'
```

代码结果：
<ul style='border: 1px solid #666'>
  <li style='color: red'>Thank you~</li>
</ul>


## 模仿jQuery实现一个API
如果要模仿jQuery，实现一个jQuery的API，那么实现方法如下：
1. 封装函数。将每个功能封装成一个函数，需要使用到时调用这个函数即可。
    ```javascript
    /* 封装添加样式的函数addClass */
    function addClass(node, className) {
      node.classList.add(className)
    }

    /* 封装设置内容的函数setText */
    function setText(node, text) {
      node.textContent = text
    }

    addClass(li, 'red')
    setText(li, 'Thank you~')
    ```
  
2. 将第一个参数（节点）提取出来变成 `node.functionName()` 的形式。

    ```javascript
    function jQuery(node){
      return {
        addClass: function(className){
          node.classList.add(className)
        },
        setText: function(text){
          node.textContent = text
        }
      }
    }

    var node = jQuery(li)
    node.addClass('red')
    node.setText('Thank you~')
    ```

3. 处理多个节点的情况。
    ```html
    <ul style='border: 1px solid #666'>
      <li>Hi</li>
      <li>Welcome</li>
    </ul>
    ```
    ```javascript
    function jQuery(nodeOrSelector){
      var elements = (typeof nodeOrSelector === 'string') ? document.querySelectorAll(nodeOrSelector) : { 0: nodeOrSelector, length: 1 }
      return {
        addClass: function(className){
          for(let i=0; i<elements.length; i++){
            elements[i].classList.add(className)
          }
        },
        setText: function(text){
          for(let i=0; i<elements.length; i++){
            elements[i].textContent = text
          }
        }
      }
    }

    var node = jQuery('li')
    node.addClass('red')
    node.setText('Thank you~')
    ```
4. 简化jQuery函数名
    ```javascript
    window.$ = jQuery;
    $('li').addClass('red')
    $('li').setText('Thank you~')
    ```
最终运行结果：
<ul style='border: 1px solid #666'>
  <li style='color: red'>Thank you~</li>
  <li style='color: red'>Thank you~</li>
</ul>


## 总结
说白了，jQuery就是一个函数，它返回一个对象，里面包含节点的属性和方法。节点可以调用这些方法来操作DOM。

PS：完整代码如下
```javascript
window.jQuery = function(nodeOrSelector){
  var elements = (typeof nodeOrSelector === 'string') ? document.querySelectorAll(nodeOrSelector) : {
    0: nodeOrSelector,
    length: 1
  }
  return {
    addClass: function(className){
      for(let i=0; i<elements.length; i++){
        elements[i].classList.add(className)
      }
    },
    setText: function(text){
      for(let i=0; i<elements.length; i++){
        elements[i].textContent = text
      }
    }
  }
}
window.$ = jQuery
```