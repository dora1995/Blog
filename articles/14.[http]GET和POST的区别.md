在公司抽空浏览了一下内部知识分享系统，记录一下~

在日常中被问到的两者区别的话，有以下这些可以回答：

- get参数通过URL传递，post放在Request body中
- get的请求比post请求快
- get请求适用于查询，post请求适用表单提交
- get请求会被浏览器主动cache，而post不会，除非手动设置
- get请求只能进行url编码，而post支持多种编码方式
- get请求参数会被完整保留在浏览器历史记录里，而post中的参数不会被保留
- get请求在URL中传送的参数是有长度限制的，而post没有限制
- 对参数的数据类型，get只接受ASCII字符，而post没有限制
- get在浏览器回退时是无害的，而post会再次提交请求
- get产生的URL地址可以被Bookmark，而post不可以
- get比post更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息

### 如果再往深一点了解

追本溯源，get、post都是HTTP协议下的方法，而HTTP又是基于TCP/IP协议的。所以说get、post方法的底层都是TCP/IP连接，做的事情的效果是一样的

给get加上request body参数，给post加上URL参数，这都是可行的

get、post都只是基于TCP的一种HTTP表现形式

在互联网中，还有另一个重要角色：不同的浏览器和服务器。多数浏览器通常都会限制URL长度在2K个字节，而大多数服务器最多处理64K大小的URL（超过的部分不处理）

如果用get服务，在Request body偷偷藏了数据，不同服务器对get的处理方式也是不同的：有些服务器会读出数据，有些服务器会忽略。所以，虽然GET可以带Request body，也不能保证一定能被接收到

### 还有一个重大区别

get产生一个TCP数据包，post产生两个TCP数据包

- 对于get方式的请求，浏览器会把HTTP Header和Data一并发送出去，服务器响应200

- 对于post，浏览器先发送Header，服务器响应100 continue，浏览器再发送Data，服务器响应200并返回数据

因为post需要两步，时间上消耗的要多一点
