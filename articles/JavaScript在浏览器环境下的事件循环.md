### 一、背景
1. JavaScript是一门单线程语言  
2. 单线程所带来的任务执行方式

### 二、同步任务与异步任务
&ensp;&ensp;单线程就意味着，所有的任务都需要排队，当前一个任务结束时，才会执行后一个任务。如果前一个任务耗时很长，后一个任务就不得不一直等待。
如果是因为计算量大，CPU忙不过来倒也正常，但是很多时候CPU是闲着的，因为IO设备（输入输出设备）很慢（比如Ajax操作从网络读取数据），不得不等着结果出来，再往下执行。

&ensp;&ensp;这时主线程完全可以不管IO设备，挂起处于等待中的任务，先运行排在后面的任务。等到IO设备返回了结果，再回过头，把挂起的任务继续执行下去。于是，所有任务可以分成两种：
1. **同步任务**
2. **异步任务**  

&ensp;&ensp;在这里用一张图来说明：


![](https://user-gold-cdn.xitu.io/2019/4/9/16a0289111b524d3?w=395&h=556&f=png&s=13875)
- 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入Event Table
- 当异步任务完成时，Event Table会将对应的回调移入Event Queue中
- JS引擎会持续不断检查主线程执行栈是否为空，当主线程内的任务全部执行完毕后，会去Event Queue读取排头第一个，进入到主线程执行
- 上述过程不断重复，形成事件循环（Event Loop）

### 三、宏任务与微任务
&ensp;&ensp;除了广义的同步任务和异步任务，会对任务有更精细的定义：
- **宏任务：整体代码、setTimeout、setInterval**
- **微任务：Promise**  

&ensp;&ensp;不同类型的任务会进入对应的Event Queue，比如setTimeout和setInterval会进入相同的Event Queue

&ensp;&ensp;第一次进入整体代码（宏任务）后，开始第一遍循环，在主线程任务全部执行完毕后，会先去读取所有的微任务进行执行，然后再到宏任务，而宏任务里面或许又包含着宏任务与微任务。以此不断循环执行

&ensp;&ensp;用一张图说明：

![](https://user-gold-cdn.xitu.io/2019/4/9/16a02a397a4dda40?w=473&h=488&f=png&s=13305)

&ensp;&ensp;举个栗子
```javascript
setTimeout(function() {
    console.log('setTimeout')
})

new Promise(function(resolve) {
    console.log('promise')
    resolve()
}).then(function() {
    console.log('then')
})

console.log('console')
```
- 这段代码会作为宏任务，进入主线程
- 先遇到setTimeout，将其放入Event Table，完成后会将其回调函数注册并放入到宏任务Event Queue
- 接下来遇到了Promise，new Promise立即执行，然后遇到console.log('promise')，立即执行。then函数放入微任务Event Queue。
- 遇到console.log('console')，立即执行
- 整体代码作为第一个宏任务执行结束，接着查看是否有微任务？我们发现了then在微任务Event Queue里面，执行
- 微任务全部执行完毕，第一轮事件循环结束，开始第二轮循环。从宏任务Event Queue发现有setTimeout对应的回调函数（多个宏任务的时候取队头），立即执行
- 检查无可执行任务，结束
-----------

在理解时候需要将上面提到的两点结合起来（同步异步 + 宏观微观），才会形成机制。在代码运行时，可以想成是以代码片段来工作的

-----------
补充：今天在刷文章的时候看到相关的题目，感觉可以加深理解一波
```javascript
    console.log('sync1')
    
    setTimeout(function (){
        console.log('setTimeout')
    }, 0)
    
    var promise = new Promise(function(resolve, reject) {
        setTimeout(function() {
            console.log('setTimeout-Promise')
        }, 0)
        console.log('promise')
        resolve()
    })
    promise.then(() => {
        console.log('promise-Then')
        setTimeout(function() {
            console.log('promise-Timeout')
        }, 0)
    })
    
    setTimeout(function() {
        console.log('lastSetTimeout')
    }, 0)
    
    console.log('sync2')
```

- 首先我们看整体代码，是第一遍同步执行，是第一个宏任务。按顺序下来一共调用了三次setTimeout（Promise中的代码也是同步执行的），调用了一次resolve，打印三次（按顺序分别是**sync1、promise、sync2**）。所以它产生出三个宏任务、一个微任务。宏任务队列按顺序有**setTimeout、setTimeout-Promise、lastSetTimeout** / 微任务队列按顺序有**promise-Then**
- 接下来，因为微任务队列不为空，先执行全部微任务。所以**promise-Then**被打印出来。然后又调用了一次setTimeout，所以promise-Timeout进入宏任务队列。此时宏任务队列按顺序为**setTimeout、setTimeout-Promise、lastSetTimeout、promise-Timeout**
- 没有微任务了，执行第二个宏任务，打印**setTimeout**，没有再产生宏任务微任务，此时微任务队列为空，所以接着执行第三个宏任务
- 同上，打印**setTimeout-Promise**
- 同上，打印**lastSetTimeout**
- 同上，打印**promise-Timeout**
- 结果为 **sync1、promise、sync2、promise-Then、setTimeout、setTimeout-Promise、lastSetTimeout、promise-Timeout**

--------

- 直到ES6，js才真正内建有直接的异步概念（Promise的引入）
- 在promise出现之前，js并没有异步，有异步的是寄主环境（文中指浏览器）
- 微任务是js引擎内部的一种机制，而宏任务是寄主环境的一种机制
- setTimeout在现设计中会有4ms的最小时间取值，就是最快也是4ms后才执行
