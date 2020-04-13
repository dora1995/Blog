
这些天在尝试开始对Vue源码的解读，一点一点去了解框架的设计以及实现思路。今天在编码时候想了有关生命周期的问题，刚好晚上就看到了相关知识。作为其中一小步记录一下

### 一、生命周期
每个Vue实例在被创建之前都要经过一系列的初始化过程。例如设置数据监听、编译模板、挂载实例到 DOM、在数据变化时更新 DOM 等  
在这个过程中会执行相应的生命周期钩子函数，给予用户机会在一些特定的场景下添加自己的逻辑代码

直接贴上官方生命周期图：

![](https://user-gold-cdn.xitu.io/2019/5/22/16ae040bfa35c9a7?w=1200&h=3039&f=png&s=50021)
可以看出生命周期是描述了一个Vue实例在创建、挂载、注销、更新的一个流程

### 二、生命周期钩子的调用
在源码中最终执行生命周期的函数是callHook方法和invokeWithErrorHandling方法，它的定义在src/core/instance/lifecycle和src/core/util/error中可以看到：
```javascript
// src/core/instance/lifecycle
export function callHook (vm: Component, hook: string) {
  pushTarget()
  const handlers = vm.$options[hook]
  const info = `${hook} hook`
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      invokeWithErrorHandling(handlers[i], vm, null, vm, info)
    }
  }
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook)
  }
  popTarget()
}

// src/core/util/error
export function invokeWithErrorHandling (
  handler: Function,
  context: any,
  args: null | any[],
  vm: any,
  info: string
) {
  let res
  try {
    res = args ? handler.apply(context, args) : handler.call(context)
    if (res && !res._isVue && isPromise(res) && !res._handled) {
      res.catch(e => handleError(e, vm, info + ` (Promise/async)`))
      res._handled = true
    }
  } catch (e) {
    handleError(e, vm, info)
  }
  return res
}
```
callHook接收的两个参数分别为**Vue实例**和要触发的**生命周期钩子名**

在触发时
- 根据hook拿到对应的**回调函数数组**(vue实例在初始化时候，其中有个过程是合并options，在该操作时会收集各个阶段的生命周期钩子函数构成对应的数组，然后将他们都挂载到实例的options中(即vm.$options)。所以在这里拿到的是一个数组。具体的可以去看一下代码)
- 如果数组有值，遍历代入invokeWithErrorHandling方法中执行(在方法中我们可以看到使用了**apply/call**，**将实例vm作为函数执行上下文传入，这也是我们在编写生命周期回调的时候，不能使用箭头函数的原因**：箭头函数的执行上下文指向定义该函数时的上下文，且无法改变，从而获取不到实例对象指向)

### 三、钩子罗列
查看Vue官网，很容易可以得到有如下钩子：
1. beforeCreate
2. created
3. beforeMount
4. mounted
5. beforeUpdate
6. updated
7. activated
8. deactivated
9. beforeDestroy
10. destroyed
11. errorCaptured

至于他们都有什么作用，官网已经写得很详细，建议看一下：[https://cn.vuejs.org/v2/api/]()

除7、8、11外，其他的在平时开发中较常用到。它们的执行顺序跟排列顺序一样

### 四、beforeCreate和created

beforeCreate和created函数都是在实例化Vue的阶段，在_init方法中执行的，它的定义在src/core/instance/init中：
```javascript
Vue.prototype._init = function (options?: Object) {
    ...
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
    ...
```
可以看到beforeCreate和created的钩子调用是在initState函数的前后，**initState的作用是初始化props、data、methods、watch、computed等属性**  

那么显然**beforeCreate的钩子函数中就不能获取到props、data中定义的值，也不能调用methods中定义的函数，而created可以**  
在这俩个钩子函数执行的时候，还没有渲染 DOM，所均访问不到DOM  

一般来说，如果组件在加载的时候需要和后端有交互，放在这俩个钩子函数执行都可以，如果是需要访问props、data等数据的话，就需要使用created钩子函数

### 五、beforeMount和mounted
beforeMount和mounted函数执行在Vue实例挂载阶段，它们的调用时机是在mountComponent函数中，定义在src/core/instance/lifecycle：
```javascript
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  ...
  callHook(vm, 'beforeMount')

  ...

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```
在执行vm._render()函数渲染 VNode 之前，执行了beforeMount钩子函数，在执行完vm._update()把 VNode patch 到真实 DOM 后，执行mounted钩子

注意，这里对mounted钩子函数执行有一个判断逻辑，vm.$vnode如果为null，则表明这不是一次组件的初始化过程，而是我们通过外部**new Vue初始化过程**

而那么对于组件，它的mounted时机在哪儿呢  
通过阅读源码我们可以发现(假装大家都阅读源码，因为上下文只对生命周期进行总结，所以深入的就不说啦~)，在组件VNode patch到DOM后，会执行invokeInsertHook函数(定义在src/core/vdom/patch.js)，会把insertedVnodeQueue里面保存的所有mounted钩子函数依次执行一遍

这一些都是题外话了，先记住Vue组件在实例化的时候会先等待子组件的实例化完成，所以insertedVnodeQueue(保存组件的mounted钩子函数的数组)的添加顺序是**先子后父**

**所以对于同步渲染的组件而言，mounted钩子函数的执行顺序是先子后父**

### 六、beforeUpdate和updated
beforeUpdate和updated的钩子函数执行时机都是在数据更新的时候  
beforeUpdate的执行时机是在 **渲染Watcher** 的before函数中，在mountComponent函数中可以看到(src/core/instance/lifecycle)：
```javascript
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  ...
  callHook(vm, 'beforeMount')

  let updateComponent
  
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  ...
  return vm
}
```
这里有个判断，也就是在组件已经mounted之后才会去调用这个钩子函数

update的执行时机是在flushSchedulerQueue函数调用的时候, 它的定义在src/core/observer/scheduler中：
```javascript
function flushSchedulerQueue () {
  currentFlushTimestamp = getNow()
  ...
  callUpdatedHooks(updatedQueue)
}

function callUpdatedHooks (queue) {
  let i = queue.length
  while (i--) {
    const watcher = queue[i]
    const vm = watcher.vm
    if (vm._watcher === watcher && vm._isMounted && !vm._isDestroyed) {
      callHook(vm, 'updated')
    }
  }
}
```
至于深入，这里就不解释啦~ (还不会

### 七、beforeDestroy和destroyed
beforeDestroy和destroyed钩子函数的执行时机在组件销毁的阶段  
beforeDestroy钩子函数的执行时机是在destroy函数执行最开始的地方，接着执行了一系列的销毁动作，包括从parent的children中删掉自身，删除watcher，当前渲染的VNode执行销毁钩子函数等，执行完毕后再调用destroy钩子函数
在$destroy的执行过程中，它又会执行vm.__patch__(vm._vnode, null)触发它子组件的销毁钩子函数，这样一层层的递归调用

**所以destroy钩子函数执行顺序是先子后父，和mounted过程一样**

### 八、activated和deactivated
activated是keep-alive组件激活时调用

deactivated是keep-alive组件停用时调用

### 九、总结
通过对整个生命周期的了解，就可以很清晰地知道可以在什么阶段做什么事，或者某一操作应该在什么阶段执行

例如在create中进行数据操作，在mounted中进行DOM完成后的操作，在destroyed进行事件解绑和功能注销

文章篇幅有点多，很多都是一些相关的衍生，只有在进行源码解读的时候才比较容易理解。**在这里最重要的是知道每个生命周期钩子的时机和作用**，其他都是浮云~