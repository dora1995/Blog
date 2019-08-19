上次写了作用域相关的知识总结，这篇文章是它的进一步提升。如果对作用域还不是很了解，可以先看一下：[JavaScript作用域相关的总结](https://github.com/dora1995/Blog/issues/4)

我翻了挺多文章，感觉他们对上下文这个词的解释都有些抽象，不如干脆就放过他的字面意义，而是记住他是如何产生的，以及作用

**当JavaScript执行一段可执行代码时，就会先建立对应的执行上下文**

执行上下文的代码会分成两个阶段进行：
- 进入执行上下文（分析）
- 代码执行（处理）

在生成的每个执行上下文中，都会有3个重要的属性：
- 变量对象
- 作用域链
- this

-----------------------------------
### 一、变量对象

>  变量对象是与执行上下文相关的数据域，存储了在上下文中定义的**变量**和**函数声明**

在**进入执行上下文**时（未代码执行），变量对象会进行初始化，包括：
1. 函数的所有形参（arguments）
2. 函数声明
3. 变量声明

举个栗子：
```javascript
function a(n, m) {
    var b = 1
    function c() {
        console.log('c')
    }
    var d = function() {
        console.log('d')
    }
    b = 2
}
a(10, 20)
```
在进入执行上下文后，此时的变量对象是这样的：
```javascript
变量对象 = {
    arguments: {
        0: 10,
        1: 20,
        length: 2
    },
    n: 10,
    m: 20,
    b: undefined,
    c: function,
    d: undefined
}
```
等到了**代码执行**时，代码会顺序执行，从而更改变量对象的值。当代码执行到最尾时，变量对象是这样的：
```javascript
变量对象 = {
    arguments: {
        0: 10,
        1: 20,
        length: 2
    },
    n: 10,
    m: 20,
    b: 1 -> 2,
    c: function,
    d: functionExpression
}
```
---------------------

### 二、作用域链
> 在函数中，当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级（词法层面）执行上下文的变量对象查找，一直找到全局上下文的变量对象。这样由多个执行上下文的变量对象构成的链表就叫做作用域链

在[作用域总结](https://juejin.im/post/5cb5e51be51d456e60003acf)中已经提过，**对于一个变量的查找方向是在作用域定义的时候就已经约定好的**

----------------------
### 三、this
JavaScript有一套不同于其他语言的对this的处理机制。在五种不同的情况下，this指向的各不相同
- **全局范围内 this - 指向全局对象**
- **函数调用 foo() - 指向全局对象**
- **方法调用 bar.foo() - 指向bar对象**
- **调用构造函数 new foo() - 指向新创建的对象**
- **显式设置 apply/call - 指向函数第一个参数**

常见误解
```javascript
Foo.method = function() {
    function test() {
        console.log(this)
    }
    test() // window
}
```
这里会被认为test函数中的this会指向Foo对象，实际上不是这样子的，而是会指向全局对象。如果想拿到Foo的内部this，可以这样写
```javascript
Foo.method = function() {
    let that = this
    function test() {
        console.log(that)
    }
    test() // Foo
```

------------------------
```javascript
let a = {
    b: function() {
        console.log(this.c)
    },
    c: 1
}
let mb = a.b
mb() // undefined (指向window)
```
另一个看起来奇怪的地方是函数别名，也就是将一个方法赋值给一个变量。上例中b 就像一个普通的函数被调用。因此，函数内的 this 不被指向到 a 对象，而是全局对象

### 四、总结
以上三个属性构成了执行上下文的作用，使得在代码执行时能够使用相关的特性