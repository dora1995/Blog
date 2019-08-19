今天抽空复习了一下web事件，将一些要点记录下来

> 在Web中, 事件在浏览器窗口中被触发并且通常被绑定到窗口内部的特定部分  
> 可能是一个元素、一系列元素、被加载到这个窗口的 HTML 代码或者是整个浏览器窗口。举几个可能发生的不同事件:
>- 用户在某个元素上点击鼠标或悬停光标。
>- 用户在键盘中按下某个按键。
>- 用户调整浏览器的大小或者关闭浏览器窗口。
>- 一个网页停止加载。
>- 提交表单。
>- 播放、暂停、关闭视频。
>- 发生错误

每个可用的事件都会有一个事件处理器，也就是事件触发时会运行的代码块。当我们定义了一个用来回应事件被激发的代码块的时候，我们说我们注册了一个事件处理器

### 一、事件绑定
主要的事件绑定方式有：
#### 1. 事件处理属性
```javascript
var btn = document.querySelector('button')
btn.onclick = function() {
    console.log('hello')
}
```
#### 2. addEventListener/removeEventListener
```javascript
var btn = document.querySelector('button')

function bgChange() {
    console.log()
}
btn.addEventListener('click', bgChange)
//btn.removeEventListener('click', bgChange)
```

----------------------------------

**两者区别：onclick(还有其他属性)赋值方式会覆盖之前设置的，而使用addEventListener可以向同一类型的元素添加多个监听器**

### 二、阻止默认行为
有时你会遇到一些情况，你希望事件不执行它的默认行为，此时可以使用 **e.preventDefault()** 来达到效果

```javascript
form.onsubmit = function(e) {
  e.preventDefault()
  console.log('hello')
}
```

### 三、事件冒泡及捕获
事件冒泡和捕捉是两种机制，主要描述当在一个元素上有两个相同类型的事件处理器被激活会发生什么

#### 1. 捕获阶段
**事件触发顺序为html元素(document)到目标元素**
- 浏览器检查元素的最外层祖先<html>，是否在捕获阶段中注册了一个onclick事件处理程序，如果是则运行它
- 然后，它移动到<html>中单击元素的下一个祖先元素，并执行相同的操作，然后是单击元素再下一个祖先元素，依此类推，直到到达实际点击的目标元素

#### 2. 冒泡阶段
**事件触发顺序为目标元素到html(document)元素**
- 浏览器检查实际点击的元素是否在冒泡阶段中注册了一个onclick事件处理程序，如果是则运行它
- 然后它移动到下一个直接的祖先元素，并做同样的事情，然后是下一个，直到它到达<html>元素

----------------------------------
**在现代浏览器中，默认情况下所有事件处理程序都在冒泡阶段进行注册**

**如果想变成为捕获阶段进行注册，可使用addEventListener(event, function, true)**

### 四、阻止冒泡
在一般情况下事件冒泡并不是我们想要的效果，而是想要它只会让当前事件处理程序运行，但事件不会在冒泡链上进一步扩大。此时可以使用**e.stopPropagation()**来达到效果

```javascript
button.onclick = function(e) {
  console.log('hello')
  e.stopPropagation()
}
```

### 五、事件委托
也叫做事件代理，是javascript中常用绑定事件的技巧。把原本需要绑定在子元素的响应事件委托给父元素，让父元素担当事件监听的职务

**原理为利用DOM元素的事件冒泡机制**

优点：
- 可以大量节省内存占用，减少事件注册
- 可以实现当新增子对象时无需再次对其绑定(动态绑定事件)

基本实现：
```html
<ul id="ul">
  <li id="1">1</li>
  <li id="2">2</li>
  <li id="3">3</li>
  <li id="4">4</li>
</ul>

```
```javascript
var ul = document.getElementById("ul")
 
ul.addEventListener("click", function (event) {
  var target = event.target;
  switch (target.id) {
    case "1": {
        console.log('i am 1')
        break;
    }
    case "2": {
        console.log('i am 2')
        break;
    }
    case "3": {
        console.log('i am 1')
        break;
    }
    default: {
        console.log('haha')
        break
    }
  }
})
```

使用事件委托时，并不是说把事件委托给的元素越靠近顶层就越好。事件冒泡的过程也需要耗时，越靠近顶层，事件的事件传播链越长，也就越耗时

如果DOM嵌套结构很深，事件冒泡通过大量祖先元素会导致性能损失