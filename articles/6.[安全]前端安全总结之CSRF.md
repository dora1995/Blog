今天来写学习CSRF的总结~

### 一、 CSRF(跨站请求伪造)

CSRF**是一种挟持用户在当前已登录的Web应用程序上执行非本意的操作的一种攻击方式**

跨域指的是**请求来源于其他网站**

伪造指的是**非用户自身的意愿**

简单的说，跨站请求伪造攻击是**攻击者通过手段欺骗用户的浏览器去访问用户曾经认证过的网站并执行一些操作（如发送邮件、发消息、甚至财产操作如转账和购买商品等）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去执行。这利用了web登录身份认证的一个漏洞：简单的身份认证只能保证请求来自用户的浏览器，但不能识别请求是用户自愿发出的**

**原理：用户在目标网站进行过登录并拿到确认后的凭证(如cookie)，攻击网站通过携带身份凭证进行对目标网站后台发送请求**


### 二、 GET CSRF
假设有这样一个场景：目标网站A(www.a.com)，恶意网站B(www.b.com)

两个网站的域名不一样，目标网站A上有一个删除文章的功能，通常是用户单击'删除文章'链接时才会删除指定的文章。这个链接是www.a.com/blog/del?id=1, id代表不同的文章。实际上就是发起一个GET请求

如果在目标网站A上存在XSS漏洞，那么可以利用这个XSS漏洞来进行攻击:
1. 使用Ajax发起一个GET请求，因为是在目标网站上，所以符合同源策略。请求参数为id=1, 请求目标地址为www.a.com/blog/del
2. 或者在目标网站A上动态创建一个标签(script, img, iframe等)，将其src指向www.a.com/blog/del?id=1, 那么攻击就会发起。实际上通过这种方式发起的请求就是一个GET请求
3. 最后欺骗用户登录目标网站A，那么攻击就会发生

如果在目标网站A上不存在XSS漏洞，那么可以利用GET CSRF进行攻击:
1. 无法使用Ajax发起GET请求。因为CSRF请求是跨域的，而Ajax有同源策略的限制
2. 可以通过在恶意网站B上静态或者动态创建img,script等标签发起GET请求。将其src属性指向www.a.com/blog/del?id=1。通过标签的方式发起的请求不受同源策略的限制(开放策略)
3. 最后欺骗已经登录目标网站A的用户访问恶意网站B，那么就会携带网站A源的登录凭证向网站A后台发起请求，这样攻击就发生了


对比CSRF和XSS攻击可以看出，CSRF攻击有以下几个关键点：
- 请求是跨域的，可以看出请求是从恶意网站B上发出的
- 通过img, script等标签来发起一个GET请求，因为这些标签不受同源策略的限制
- 发出的请求是身份认证后的

### 三、POST CSRF

假如目标网站A上有发表文章的功能，那么我们就可以动态创建form标签，然后修改文章的题目。

在网站B中：
```javascript
function setForm () {
    var form = document.createElement('form')
    form.action = 'www.a.com/blog/article/update'
    form.methods = 'POST'
    var input = document.createElement('input')
    input.type = 'text'
    input.value = 'csfr攻击啦！'
    input.id = 'title'
    form.appendChild(input)
    document.body.appendChild(form)
    form.submit()
}
setForm()
```
上面代码可以看出，动态创建了form表单，然后调用submit方法，就可以通过跨域的伪造请求来实现修改目标网站A的某篇文章的标题了

### 四、CSRF的危害
CSRF的危害：
- 利用用户登录态
- 用户不知情
- 盗取用户资金
- 冒充用户完成操作或修改数据

### 五、CSRF的防范
- 禁止第三方网站带Cookies

    设置Cookies中的same-site属性，使之只能在同网站下使用
- 检测请求头中的Referer字段

    从目标网站A和恶意网站B发出的请求中，请求头唯一的不同就是Referer字段
    
    以上面删除文章功能为例，在目标网站A中的Referer字段为http://www.a.com/blog/，而恶意网站B中的Referer字段为http://www.b.com/csrf.html。根据Referer字段与Host字段在同一域名下的规则，可以检测Referer字段值，如果发现其与Host值不在同一域名下，这时候服务器就能够识别出恶意的访问了

- 添加检验token

    由于CSRF的本质在于攻击者欺骗用户去访问自己设置的地址，所以如果要求在访问敏感数据请求时，要求用户浏览器提供不保存在cookie中，并且攻击者**无法伪造的数据**作为校验，那么攻击者就无法再执行CSRF攻击。这种数据通常是表单中的一个数据项。服务器将其生成并附加在表单中，其内容是一个随机数。即<input type="hidden" name="_csrf_token" value="xxxx">的形式

    当客户端通过表单提交请求时，这个随机数也一并提交上去以供校验。正常的访问时，客户端浏览器能够正确得到并传回这个随机数，而通过CSRF传来的欺骗性攻击中，攻击者无从事先得知这个随机数的值，服务器端就会因为校验token的值为空或者错误，拒绝这个可疑请求