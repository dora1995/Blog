前端安全常见的攻击方式有两种：**XSS**(CRoss Site Scripting **跨站脚本攻击**)、**CSRF**(Cross-site request forgery  **跨域请求伪造**)

为了控制篇幅~今天先写XSS

### 一、 XSS(跨站脚本攻击)
**原理：程序+数据=结果，然而数据里面包含了程序，导致结果运行，而不单单是展示**

XSS可以做：
- 获取页面数据
- 获取cookies
- 劫持前端逻辑
- 发送请求
- 偷取网站任意数据
- 偷取用户资料
- 偷取用户密码和登录态
- 欺骗用户
- 等等

XSS攻击可能的注入点(均是写入可执行脚本的方式)：
- HTML节点内容，存在读取可输入数据
- HTML属性，存在读取可输入数据
- JavaScript代码，存在由后台注入的变量或用户输入的信息
- 富文本

XSS是发生在**目标用户的浏览器**层面上的，当渲染DOM树的过程成发生了不在预期内执行的JS代码时，就发生了XSS攻击。
跨站脚本的重点不在‘跨站’上，而在于‘**脚本**’上。**大多数XSS攻击的主要方式是嵌入一段远程或者第三方域上的JS代码**。**实际上是在目标网站的作用域下执行了这段js代码**

### 二、 反射型XSS
反射型XSS，也叫非持久型XSS，**是指发生请求时，XSS代码出现在请求URL中，作为参数提交到服务器，服务器解析并响应。响应结果中包含XSS代码，最后浏览器解析并执行** (经过服务器，不经过数据库)

从概念上可以看出，**反射型XSS代码是首先出现在URL中的，然后需要服务端解析，最后需要浏览器解析之后XSS代码才能够攻击**

举个栗子：
```html
<textarea name="txt" id="txt" cols="80" rows="10">
<button type="button" id="test">点我</button>
```
```javascript
let test = document.querySelector('#test')
test.addEventListener('click', function () {
    // 1. 发送一个GET请求
    let url = `/test?test=${txt.value}`   
    let xhr = new XMLHttpRequest()
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            if ((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304) {
            // 3. 客户端解析JSON并执行
            let str = JSON.parse(xhr.responseText).test
            let node = `${str}`
            document.body.insertAdjacentHTML('beforeend', node)
        } else {
            console.log('error', xhr.responseText)
        }
    }
  }
  xhr.open('GET', url, true)
  xhr.send(null)
}, false)
```
```javascript
var express = require('express');
var router = express.Router();

router.get('/test', function (req, res, next) {
    // 2. 服务端解析成JSON后响应
    res.json({
        test: req.query.test
    })
})
```
启一个web服务器，然后设置请求接口。通过Ajax的GET请求将参数发往服务器，服务器解析成json后响应。将返回的数据解析后显示到页面上(没有对返回的数据进行解码和过滤等操作)

此时我们往textarea输入一段带有攻击目的的文本：'\<img src="null" onerror='alert(document.cookie)' \/\>'

然后点击按钮，一个XSS攻击就发生了，页面会alert出cookie信息

**实际上，我们只是模拟攻击，通过alert获取到了个人的cookie信息。但是如果是黑客的话，他们会注入一段第三方的js代码，然后将获取到的cookie信息存到他们的服务器上。这样的话黑客们就有机会拿到我们的身份认证做一些违法的事情了**

### 三、 储存型XSS
存储型XSS，也叫持久型XSS，**主要是将XSS代码发送到服务器(不管是数据库、内存还是文件系统等)，然后在下次请求页面的时候就不用带上XSS代码了** (经过服务器，经过数据库)

最典型的就是留言板XSS：**用户提交了一条包含XSS代码的留言到数据库，当目标用户查询留言时，那些留言的内容会从服务器解析之后加载出来，浏览器发现有XSS代码，就当做正常的HTML和JS解析执行。XSS攻击就发生了**

### 四、XSS危害
- 通过document.cookie盗取cookie
- 使用js或css破坏页面正常的结构与样式
- 流量劫持（通过访问某段具有window.location.href定位到其他页面:\<script\>window.location.href="http://www.baidu.com";\<\/script\>）
- Dos攻击：利用合理的客户端请求来占用过多的服务器资源，从而使合法用户无法得到服务器响应
- 利用iframe、frame、XMLHttpRequest或上述Flash等方式，以（被攻击）用户的身份执行一些管理动作，或执行一些一般的如发微博、加好友、发私信等操作
- 利用可被攻击的域受到其他域信任的特点，以受信任来源的身份请求一些平时不允许的操作，如进行不当的投票活动

### 五、XSS防御
从以上的反射型和DOM XSS攻击可以看出，**我们不能原样的将用户输入的数据直接存到服务器，需要对数据进行一些处理**。以上的代码出现的一些问题如下：

- 没有过滤危险的DOM节点。如具有执行脚本能力的script, 具有显示广告和图片的img, 具有改变样式的link、style, 具有内嵌页面的iframe, frame等元素节点
- 没有过滤危险的属性节点。如事件, style, src, href等
- 没有对用户输入进行处理和过滤
- 没有对cookie设置httpOnly

解决方法如下：

- **对cookie的保护**

    对重要的cookie设置httpOnly，防止客户端通过document.cookie读取cookie。服务端可以设置此字段

- **对用户输入数据的处理**

    - 编码：不能对用户输入的内容都保持原样，对用户输入的数据进行字符实体编码转义
    - 解码：原样显示内容的时候必须解码，不然显示不到内容了
    - 过滤：把输入的一些不合法的东西都过滤掉，从而保证安全性。如移除用户上传的DOM属性，如onerror，移除用户上传的Style节点，iframe, script节点等
    
    
    
