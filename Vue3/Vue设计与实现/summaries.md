[toc]

### 框架设计预览

#### 权衡的艺术

> 框架设计里到处都体现了权衡的艺术

**命令式框架关注过程，声明式框架关注结果**

声明式代码的性能不优于命令式代码的性能，Vue.js内部实现一定是**命令式**的，暴露给用户的却更加**声明式**

<font size="3">**声明式代码的更新性能消耗 = 找出差异的性能消耗 + 直接修改的性能消耗**，所谓的**虚拟DOM**，就是为了**最小化**找出差异这一步的性能消耗而出现的</font>

对比innerHTML、虚拟DOM、原生javascript(指createElement)来更新页面，可发现在心智负担方面虚拟DOM最小，原生js最大；在性能方面，原生js性能最好，innerHTML性能最差；在可维护性方面，虚拟DOM可维护性最强，原生js可维护性最差；所以需要做到，既声明式的描述UI，又要具备原生js的性能。

在结合**心智负担、可维护性**等因素综合考虑，虚拟DOM是个不错的选择

纯运行时的框架没有编译的过程，无法分析用户提供的内容；纯编译时的框架有损灵活性；Vue.js 3是一个**编译时 + 运行时**的框架，在保持灵活性的基础上，可以通过编译手段分析用户提供的内容，从而进一步提升性能。

#### 框架设计的核心要素

在框架设计与开发的时候，提供**友好的警告信息**至关重要，有助于开发者快速定位问题。为了框架体积不受警告信息的影响，需要；利用`Tree- Shaking`机制，配合构建工具(如`rollup.js`)预定义常量的能力，比如预定义`_DEV_`常量，从而实现只在开发环境中打印警告信息，生产环境中则不会包含这些影响开发体验的代码，**控制代码体积**。

`Tree- Shaking`是一种排除dead code的机制。

对于**框架的输出产物**，不同类型的产物满足不同的需求，为了让用户能够通过<script>标签直接引用并使用，我们需要输出 IIFE格式的资源，即立即调用的函数表达式。为了让用户能够通过 <script type="module"> 引用并使用，我们需要输出 ESM格式的资源。这里需要注意的是，ESM 格式的资源有两种：用于浏览器的 `esm-browser.js` 和用于打包工具的`esm-bundler.js` 它们的区别在于对预定义常量`_DEV_`的处理，前者直接将`_ DEV_ `常量替换为字面量 true 或false(从而控制需要`Tree-Shaking`的代码)，后者则将`_DEV_ `常量替换为` process.env.NODE_ ENV !=='production'` 语句。

框架的错误处理做得好坏直接决定了用户应用程序的健壮性，同时还决定了用户开发应月处理错误的心智负担。框架需要为用户提供统一的**错误处理接口**，这样用户可以通过注册自定的错误处理两数来处理全部的框架异常。

框架还需提供更加良好的`TypeScript`**类型支持**



#### Vue.js设计思路

本章中，介绍了声明式地描述 UI 的概念。vuejs 是一个声明式的框架，声明式的好处在于，它直接表达结果，用户不需要关注过程。vuejs采用模板的方式来描述UI，但它同样支持使用虛拟DOM 来描述 UI。虛拟DOM 要比模板更加灵活，但模板要比虚拟DOM 更加直观。

**渲染器**的作用是，把**虚拟 DOM** 对象渲染为真实DOM 元素。它的工作原理是递归地遍历虚拟 DOM 对象，并调用原生 DOM API 来完成真实DOM 的创建。渲染器的精髓在于后续的更新，它会通过 <font size="3">**DIFF算法**</font>找出变更点，并且只会更新需要更新的内容。

组件其实就是**一组虛拟DOM元素的封装**，它可以是一个返回虛拟DOM 的函数，也可以是一个对象，但这个对象下必须要有一个函数，用来产出组件要渲染的虚拟 DOM。渲染器在渲染组件时，会先获取组件要渲染的内容，即执行组件的渲染函数并得到其返回值，称之为 subtree，最后再递归地调用渲染器将 subtree 渲染出来即可。

Vuejs 的模板会被一个叫作**编译器**的程序编译为渲染函数，编译器、渲染器都是 Vue.js 的核心组成部分，它们共同构成一个有机的整体，不同模块之间互相配合，进一步提升框架性能。

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20230215235745491.png" alt="image-20230215235745491" style="zoom:50%;" />



### 响应系统

#### 响应系统的作用与实现

```js
const obj = {text: 'cyk'}
function effect() {
  document.body.innerText = obj.text
}
```

希望值变化后，副作用函数effect会重新执行，更新数据，为了实现响应式数据，可以先发现两点线索：

- 当副作用函数执行时会触发obj.text的**读取**操作
- 当修改obj.text时，会触发字段obj.text的**设置**操作

初始想法是当读取obj.text字段时，可以将副作用函数存储到一个“桶”里，这样，当设置obj.text时，再将该副作用函数取出来执行即可。

在Vue3中，使用代理对象`Proxy`来拦截一个对象属性的读取和设置操作

初始版本直接通过名字(effect)来获取副作用函数，很不方便，后来完善后：

- 当**读取**操作发生时，将副作用函数收集到桶里
- 当**设置**操作发生时，从桶中取出副作用函数并执行

通过注册全局变量来存储被注册的副作用函数，解决了上述问题，不过也发现，由于**没有在副作用函数与被操作的目标字段之间建立明确的联系**，导致无论读取的是什么属性，都会收集副作用函数；无论设置的是什么属性，都会把桶里的副作用函数取出并执行。这需要重新设计桶的结构，不能再简单的使用一个Set作为桶了。

```js
function effect(function effectFn(){
  document.body.innerText = obj.text
})
```

此段代码存在三个角色

- 被操作的代理对象obj
- 被操作的字段名text
- 使用effect函数注册的副作用函数effectFn

若用target代表一个代理对象所代理的原始对象，用key来表示被操作的字段名，用effectFn来表示被注册的副作用函数，那么可以建立如下的关系

```shell
--target
     --key
        --effectFn
```

选择`WeakMap`来存储副作用函数和目标字段的关系，选择`WeakMap`来存储是因为WeakMap对key是弱引用，若用户侧对代码对target没有任何引用，target会被垃圾回收器回收。

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20230217001550046.png" alt="image-20230217001550046" style="zoom:30%;" align='left'/>

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20230217002243702.png" alt="image-20230217002243702" style="zoom:50%;" align="left"/>

```js
// 存储副作用函数的桶
const bucket = new WeakMap()
// 原始数据
const data = { text: 'hello world' }

const obj = new Proxy(data, {
  // 拦截对象属性的读取
  get(target, key) {
    // 收集依赖，将副作用函数添加进桶
    track(target, key)
    return target[key]
  },
  // 拦截对象属性的设置
  set(target, key, newVal) {
    target[key] = newVal
    // 触发变化，将副作用函数从桶中取出并执行
    trigger(target, key)
  },
})

//在get拦截函数内调用track函数追踪变化，收集依赖
function track(target, key) {
  if (!activeEffect) return
  // 根据target从桶里拿到depsMap，值是Map(key --> effects)
  let depsMap = bucket.get(target)
  // 若不存在，创建一个Map并与target关联
  if (!depsMap) bucket.set(target, (depsMap = new Map()))
  // 根据key从depsMap里拿到deps，是Set类型，存放着与key相关的副作用函数effects
  let deps = depsMap.get(key)
  // 若不存在，创建一个Set并与key关联
  if (!deps) depsMap.set(key, (deps = new Set()))
  deps.add(activeEffect)
}

//在set拦截函数内调用trigger函数触发变化
function trigger(target, key) {
  const depsMap = bucket.get(target)
  if (!depsMap) return
  const effects = depsMap.get(key)
  effects && effects.forEach((fn) => fn())
}

// 用一个全局变量存储当前激活的 effect 函数
let activeEffect = undefined
function effect(fn) {
  // 当调用 effect 注册副作用函数时，将副作用函数复制给 activeEffect
  activeEffect = fn
  // 执行副作用函数
  fn()
}

effect(() => {
  console.log('effect run')
  document.body.innerText = obj.text
})

setTimeout(() => {
  obj.text = 'hello vue3'
}, 1000)
```

以上便是一个相对完善的响应式系统

如此就完备了吗？ 当然不是，首先便是**分支切换**

```js
 const data = { ok: true, text: 'hello world' }
 const obj = new Proxy(data, { /* ... */ })

 effect(function effectFn() {
   document.body.innerText = obj.ok ? obj.text : 'not'
 })
```

根据上述代码，可知由于字段obj.ok的初始值为`true`，所以副作用函数首先会被字段data.ok和data.text所对应的依赖集合收集，当obj.ok的值改为`false`时，会触发副作用函数重新执行，由于此时obj.text不会被执行，所以理想状态下副作用函数effectFn不应该被obj.text所对应的依赖集合收集，但目前我们的代码实现不了这一点，**会产生遗留的副作用函数**，因为我们只要修改了obj.text的值，effect都会重新执行，即使innerText的值不需要变化。

解决这个问题也很简单，只需在每次执行副作用函数后，将它从所有相关的依赖集合中删除，因此需要重新设计部分函数。

在`track`中完成对依赖集合的收集

```js
function track(target, key) {
  /* if (!activeEffect) return // 没有正在执行的副作用函数 直接返回 
  let depsMap = bucket.get(target)
  if (!depsMap) {
    bucket.set(target, (depsMap = new Map()))
  }
  let deps = depsMap.get(key)
  if (!deps) {
    depsMap.set(key, (deps = new Set()))
  } */
  // deps就是与当前副作用函数相关的依赖集合
  deps.add(activeEffect)
  // 将该依赖集合存储到activeEffect.deps中
  activeEffect.deps.push(deps)
}
```

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20230219225137550.png" alt="image-20230219225137550" style="zoom:40%;" align="left" />

有了这个联系后，我们就可以在每次副作用函数执行时，获取到所有相关联的依赖集合，进而将副作用函数从依赖集合中删除

```js
// 用一个全局变量存储当前激活的 effect 函数
let activeEffect
function effect(fn) {
  const effectFn = () => {
    cleanup(effectFn)
    // 当调用 effect 注册副作用函数时，将副作用函数复制给 activeEffect
    activeEffect = effectFn
    fn()
  }
  // activeEffect.deps 用来存储所有与该副作用函数相关的依赖集合
  effectFn.deps = []
  // 执行副作用函数
  effectFn()
}

function cleanup(effectFn) {
  for (let i = 0; i < effectFn.deps.length; i++) {
    const deps = effectFn.deps[i]
    deps.delete(effectFn)
  }
  effectFn.deps.length = 0
}
```

但此时依然没完，问题出现在`trigger`函数中，在该函数内部会遍历effect集合并执行副作用函数，由于是个Set集合，根据语言规范描述，当调用forEach遍历Set集合时，如果一个值已经被访问过了，但该值被删除并重新添加到了集合，若此时forEach没有遍历结束，那么该值会重新被访问，所以会出现类似`set.delete(1)  set.add(1)`这样的代码，会导致无限执行，解决方法是构造一个新的Set集合并遍历。

```js
 function trigger(target, key) {
  const depsMap = bucket.get(target)
  if (!depsMap) return
  const effects = depsMap.get(key)

  const effectsToRun = new Set()
  effects && effects.forEach((effectFn) => effectsToRun.add(effectFn))
  effectsToRun.forEach((effectFn) => effectFn())
  // effects && effects.forEach(effectFn => effectFn())
}
```

**嵌套的effect：**

对于Vue.js来说，渲染函数是会运行在一个effect()函数中的，所以当组件发生嵌套时，也会发生effect()嵌套，上述代码目前不支持effect()嵌套，问题出在我们实现的effect函数与activeEffect上，我们是用的全局变量activeEffect来存储通过effect函数注册的副作用函数，这意味着当副作用函数发生嵌套时，内层副作用函数的执行会覆盖activeEffect的值，这时如果再有响应式数据执行依赖收集，即使这个响应式数据是在外层副作用函数中读取的，它们收集到的副作用函数也都会是内层副作用函数。

我们需要一个**副作用函数栈effectStack**，当副作用函数执行时，会被压入栈，执行完毕后会被弹出，并始终让activeEffect指向栈顶顶副作用函数，这样就能做到一个响应式数据只会收集直接读取其值的副作用函数。

```js
// 用一个全局变量存储当前激活的 effect 函数
let activeEffect
const effectStack = []
function effect(fn) {
  const effectFn = () => {
    cleanup(effectFn)
    // 当调用 effect 注册副作用函数时，将副作用函数赋值给 activeEffect
    activeEffect = effectFn
    // 调用前将当前副作用函数压入栈中
    effectStack.push(effectFn)
    fn()
    // 副作用函数执行后，将当前副作用函数从栈中弹出
    effectStack.pop()
    activeEffect = effectStack[effectStack.length - 1]
  }
  // activeEffect.deps 用来存储所有与该副作用函数相关的依赖集合
  effectFn.deps = []
  // 执行副作用函数
  effectFn()
}
```

**避免无限递归循环**

实现一个完善的响应式系统需要考虑诸多细节，无限递归循环自然要考虑在内

```js
const data = {foo: 1}
const obj = new Proxy(data, {/*...*/})

effect(() => obj.foo++)
```

该注册的副作用函数的自增操作，会引起栈溢出，自增操作分开来看实际上是这样的：

```js
obj.foo = obj.foo + 1
```

在这个语句中，既会读取obj.foo的值，又会设置obj.foo的值，代码执行流程如下：首先读取obj.foo，触发`track`操作，将当前副作用函数收集到桶中，接着+1再赋值给obj.foo，会触发`trigger`操作，把桶里的副作用函数取出来执行，但问题是该副作用函数正在执行中，还没执行完毕就要开始下次执行，会导致无限递归地调用自己，爆栈。

分析问题后发现，读取和设置操作是在同一个副作用函数里的，此时无论时`track`还是`trigger`时收集的副作用函数，都是activeEffect。所以我们可以在trigger动作发生的时候增加守卫条件：**如果 trigger 触发执行的副作用函数与当前正在执行的副作用函数相同，则不触发执行**。

```js
function trigger(target, key) {
   const depsMap = bucket.get(target)
   if (!depsMap) return
   const effects = depsMap.get(key)

   const effectsToRun = new Set()
   effects && effects.forEach(effectFn => {
     // 如果 trigger 触发执行的副作用函数与当前正在执行的副作用函数相同，则不触发执行
     if (effectFn !== activeEffect) {  // 新增
       effectsToRun.add(effectFn)
     }
   })
   effectsToRun.forEach(effectFn => effectFn())
 }
```

**调度执行**

可调度性是响应系统非常重要的特性，所谓可调度，即是当trigger动作触发副作用函数重新执行时，有能力决定副作用函数执行的时机、次数以及方式。

可以为effect函数设计一个选项参数options，允许用户指定调度器：

```js
effect(
  () => {
    // ......
  },
  // options
  {
    scheduler: (fn) => {
      console.log('scheduler run')
    },
  }
)
```

同时在`effect`函数内部需要把options选项挂载到对应的副作用函数上：

```js
function effect(fn, options = {}) {
  const effectFn = () => {
    // ......
  }
  // 将 options 挂载到 effectFn 上
  effectFn.options = options
  // activeEffect.deps 用来存储所有与该副作用函数相关的依赖集合
  effectFn.deps = []
  // 执行副作用函数
  effectFn()
}
```

有了调度函数，在`trigger`函数中触发副作用函数重新执行时，就可以直接调用用户传递的调度器函数，将控制权交给用户：

```js
function trigger(target, key) {
  // .......
  effectsToRun.forEach((effectFn) => {
    // 如果副作用函数有 调度器，则调用调度器，并将副作用函数作为参数传入
    if (effectFn.options.scheduler) {
      effectFn.options.scheduler(effectFn)
    } else {
      effectFn()
    }
  })
}
```

编写如下的调度器函数，原本需要打印1 2 3 4的副作用函数会跳过中间的过程直接打印1 4，Vue.js里，连续**多次修改响应式数据只会触发一次更新**，也是实现了个类似思路的更加完善的调度器。

```js
const data = { foo: 1 }

// 若同一个副作用函数被多次添加，Set的特性会使其去重
const jobQueue = new Set()
// 用一个 Promise 来保证 jobQueue 中的副作用函数是异步执行的
const p = Promise.resolve()

// 用一个变量 isFlushing 来标识是否正在刷新队列
let isFlushing = false
// 该函数作用是，在一个周期内，只执行一次 jobQueue 中的副作用函数
function flushJob() {
  if (isFlushing) return
  isFlushing = true
  p.then(() => {
    jobQueue.forEach((job) => job())
  }).finally(() => {
    isFlushing = false
  })
}

effect(
  () => {
    console.log(obj.foo) // 1  3
  },
  {
    scheduler(fn) {
      // 每次调度时，将副作用函数添加到 jobQueue 中
      jobQueue.add(fn)
      // 调用 flushJob 函数，执行将 jobQueue 中的副作用函数
      flushJob()
    },
  }
)

obj.foo++
obj.foo++
obj.foo++
```

##### 计算属性computed与lazy

综合以上那些内容，可以实现Vue.js一个非常有特色有重要的能力-**计算属性**；在实现计算属性前，先来关注懒执行的effect，即不立即执行传递给effect的副作用函数。

修改`effect`的函数实现，若存在lazy选项，不会立即执行副作用函数，并将副作用函数作为返回值返回。

```js
function effect(fn, options = {}) {
  const effectFn = () => {
    // ......
  }
  // .....
  // 如果传入 lazy 选项，则不会立即执行副作用函数
  if (!options.lazy) {
    effectFn()
  }
  return effectFn
}
```

可以手动执行

```js
const effectFn = effect(() => console.log(obj.foo), { lazy: true })
effectFn() // 手动执行
```

仅仅能手动执行，作用不大，可以将传递给effect的函数作为一个getter，那么这个getter函数可以返回任何值。

```js
const effectFn = effect(() => obj.foo + 1, { lazy: true })
// 得到getter的返回值
const value = effectFn()
```

只需对`effect`进行小修改便可：

```js
function effect(fn, options = {}) {
  const effectFn = () => {
    // .....
    // 存储fn的执行结果
    const res = fn()
    // .....
    return res
  }
  // .....
  // 如果传入 lazy 选项，则不会立即执行副作用函数
  if (!options.lazy) {
    effectFn()
  }
  return effectFn
}
```

接下来便可开始实现计算属性了：

```js
function computed(getter) {
  // 传入的 getter 函数作为副作用函数
  const effectFn = effect(getter, { lazy: true })

  const obj = {
    // 当读取value时才执行effectFn
    get value() {
      return effectFn()
    },
  }
  return obj
}
```

不过上面👆实现的计算属性只做到了懒计算，做不到对值进行缓存，多次访问sunRes.value会导致effectFn多次计算，即使依赖的属性值obj.foo + obj.bar并没有变化。

```js
const sumRes = computed(() => obj.foo + obj.bar)

console.log(sumRes.value)
```

为了解决这个问题，就需要在实现computed函数时，添加对值进行缓存的功能。

```js
function computed(getter) {
  let value
  let dirty = true

  const effectFn = effect(getter, {
    lazy: true,
    // 添加调度器，在调度器中将 dirty 置为 true
    scheduler() {
      if (!dirty) {
        dirty = true
        // 当计算属性依赖的响应式数据发生变化时，手动调用trigger出触发响应
        trigger(obj, 'value')
      }
    },
  })

  const obj = {
    get value() {
      if (dirty) {
        value = effectFn()
        dirty = false
      }
      // 读取value时，手动调用track触发依赖收集
      track(obj, 'value')
      return value
    },
  }

  return obj
}
```

第一次访问value值后，变量dirty就会设置为false，会导致即使修改了obj.foo的值，都不会重新计算了，所以就需要在调度器里重置dirty为true，依赖值发生变化时才会重新计算。调度器函数在这里只有依赖值改变了才会触发，因为如果没有改变就不会走set拦截，不会被trigger调用副作用函数。

手动trigger和track的原因是，对于计算属性内部的getter函数来说，它里面访问的响应式数据只会把computed内部的effect收集为依赖；而把计算属性用于另一个effect时，会发生effect嵌套，外层的effect不会被内层effect中的响应式数据收集，所以需要手动调用track来追踪，手动调用trigger来响应。

```js
effect(() => {
  console.log(sumRes.value)
})
```

即对于如下代码来说：

```js
effect(function effectFn) {
  console.log(sumRes.value)
}
```

会建立这样的联系：

```bash
--computed(obj)
     --value
        --effectFn
```

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20230226214220706.png" alt="image-20230226214220706" style="zoom:50%;" />

关于该依赖关系的理解：[issue--77](https://github.com/HcySunYang/code-for-vue-3-book/issues/77)

##### watch

watch的本质就是观测一个响应式数据，当数据发生变化时通知并执行相应的回调函数，本质上就是利用了effect以及options.scheduler选项。

`traverse()`函数实际上就是对一个对象做深层递归遍历，因为遍历过程中就是对一个子属性的访问，会触发它们的 getter 过程，这样就可以track收集到相应的依赖，这样，当深层次对象变化的时候，就会trigger相应的副作用函数更新。

在实现获取旧值和新值中，主要依靠了lazy选项创建了懒执行的effect，手动调用effectFn函数得到的返回值就是旧值，当变化发生并触发scheduler调度函数执行时，会重新调用effectFn函数并得到新值，并通过回调函数传递出来。

```js
function watch(source, cb) {
  let getter
  // 如果 source 是一个函数，则直接将 source 赋值给 getter
  if (typeof source === 'function') {
    getter = source
  } else {
    // 调用traverse递归读取source中的所有属性
    getter = () => traverse(source)
  }

  let oldValue, newValue
  const effectFn = effect(() => getter, {
    lazy: true,
    scheduler() {
      newValue = getter()
      cb(newValue, oldValue)
      oldValue = newValue
    },
  })
  oldValue = effectFn()
}

function traverse(value, seen = new Set()) {
  if (typeof value !== 'object' || value === null || seen.has(value)) return
  seen.add(value)
  for (const k in value) {
    traverse(value[k], seen)
  }
  return value
}
```

**立即执行的watch与回调执行时机**

实现立即执行的watch，可以将scheduler调度函数封装成通用函数，分别在初始化和变更时执行它

flush值为post时，代表调度函数需要将副作用函数放到一个微任务队列中，并等待DOM更新结束后再执行

```js
function watch(source, cb, options = {}) {
  let getter
  if (typeof source === 'function') {
    getter = source
  } else {
    getter = () => traverse(source)
  }

  let oldValue, newValue
  // 封装scheduler调度函数是为了控制执行时机
  const job = () => {
    newValue = effectFn()
    cb(oldValue, newValue)
    oldValue = newValue
  }

  const effectFn = effect(
    // 执行 getter
    () => getter(),
    {
      lazy: true,
      scheduler: () => {
        // 在调度函数中判断flush是否为post，如果是则将调度函数放入微任务队列中
        if (options.flush === 'post') {
          const p = Promise.resolve()
          p.then(job)
        } else {
          job()
        }
      },
    }
  )
  // 如果传入 immediate 选项，则立即执行job，从而触发副作用函数
  if (options.immediate) {
    job()
  } else {
    oldValue = effectFn()
  }
}
```

**过期的副作用**

过期的副作用函数会导致竞态问题，即连续执行两次watch的回调，调用两次接口，第二次接口可能比第一次接口返回值的速度要快，导致原本应该获得的第二次的返回值被第一次的返回值给覆盖。

Vue.js里，watch函数的回调函数接受第三个参数onInvalidate [watch api](https://cn.vuejs.org/api/reactivity-core.html#watch)，用于注册副作用清理的回调函数。该回调函数会在副作用下一次重新执行前调用，可以用来清除无效的副作用，例如等待中的异步请求。

```js
  let oldValue, newValue
  // cleanup用来存储用户注册的过期回调
  let cleanup
  function onInvalidate(fn) {
    cleanup = fn
  }

  const job = () => {
    newValue = effectFn()
    // 在调用回调函数cb前，先调用过期回调
    if (cleanup) {
      cleanup()
    }
    cb(oldValue, newValue, onInvalidate)
    oldValue = newValue
  }
```

第一次修改是立即执行的，这会导致 watch 的回调函数执行。由于我们在回调函数内调用了 onInvalidate，所以会注册一个过期回调，接着发送请求A。假设请求 A 需要 1000ms 才能返回结果，而我们在 200ms 时第二次修改了 obj.foo 的值，这又会导致 watch 的回调函数执行。这时要注意的是，在我们的实现中，每次执行回调函数之前要先检查过期回调是否存在，如果存在，会优先执行过期回调。由于在 watch 的回调函数第一次执行的时候，我们已经注册了一个过期回调，所以在watch 的回调函数第二次执行之前，会优先执行之前注册的过期回调，这会使得第一次执行的副作用函数内闭包的变量 expired 的值变为 true，即副作用函数的执行过期了。于是等请求 A 的结果返回时，其结果会被抛弃，从而避免了过期的副作用函数带来的影响，

```js
watch(
  () => obj.foo,
  async (newVal, oldVal, onInvalidate) => {
    let valid = true
    onInvalidate(() => {
      valid = false
    })
    const res = await fetch()

    if (!valid) return

    finallyData = res
    console.log(finallyData)
  },
)
```

#### 总结

​    首先介绍了副作用函数和响应式数据的概念，以及它们之间的关系。一个响应式数据最基本的实现依赖于对“读取”和 “设置”操作拦截，从而在副作用函数与响应式数据之间建立联系。当“读取”操作发生时，我们将当前执行的副作用函数存储到“桶”中；当“设置”操作发生时，再将副作用函数从“桶”里取出并执行。这就是响应系统的根本实现原理。

​    接着，我们实现了一个相对完善的响应系统。**使用 WeakMap 配合Map 构建了新的“桶”结构**，从而能够在响应式数据与副作用函数之间建立更加精确的联系。同时，我们也介绍了 WeakMap 与 Map 这两个数据结构之间的区别。WeakMap 是弱引用的，它不影响垃圾回收器的工作。当用户代码对一个对象没有引用关系时，WeakMap 不会阻止垃圾回收器回收该对象。

​    我们还讨论了**分支切换导致的冗余副作用**的问题，这个问题会导致副作用函数进行不必要的更新。为了解决这个问题，我们需要在每次副作用函数重新执行之前，清除上一次建立的响应联系，而当副作用函数重新执行后，会再次建立新的响应联系，新的响应联系中不存在冗余副作用问题，从而解决了问题。但在此过程中，我们还遇到了**遍历 Set 数据结构导致无限循环**的新问题，该问题产生的原因可以从ECMA 规范中得知，即“在调用 forEach 遍历 Set 集合时，如果一个值已经被访问过了，但这个值被删除并重新添加到集合，如果此时forEach 遍历没有结束，那么这个值会重新被访问。”决方案是建立一个新的 Set 数据结构用来遍历。

​    然后，我们讨论了关于**嵌套的副作用函数**的问题。在实际场景中，嵌套的副作用函数发生在组件嵌套的场景中，即父子组件关系。这时为了避免在响应式数据与副作用函数之间建立的响应联系发生错乱，我们需要使用副作用函数栈来存储不同的副作用函数。当一个副作用函数执行完毕后，将其从栈中弹出。当读取响应式数据的时候，被读取的响应式数据只会与当前栈顶的副作用函数建立响应联系，从而解决问题。而后，我们遇到了副作用函数无限递归地调用自身，导致栈溢出的问题。该问题的根本原因在于，对响应式数据的读取和设置操作发生在同一个副作用函数内。解决办法很简单，如果 trigger触发执行的副作用函数与当前正在执行的副作用函数相同，则不触发执行。

​    随后，我们讨论了**响应系统的可调度性**。所谓可调度，指的是当trigger 动作触发副作用函数新执行时，有能力决定副作用函数执行的时机、次数以及方式。为了实现调度能力，我们为 effect 函数增加了第二个选项参数，可以通过 scheduler 选项指定调用器，这样用户可以通过调度器自行完成任务的调度。我们还讲解了如何通过调度器实现任务去重，即通过一个微任务队列对任务进行缓存，从而实现去重。

​    而后，我们讲解了**计算属性**，即 computed。计算属性实际上是一个懒执行的副作用函数，我们通过 lazy 选项使得副作用函数可以懒执行。被标记为懒执行的副作用函数可以通过手动方式让其执行。利用这个特点，我们设计了计算属性，当读取计算属性的值时，只需要手动执行副作用函数即可。当计算属性依赖的响应式数据发生变化时，会通过 scheduler 将 dirty 标记设置为 true，代表“脏”。这样，下次读取计算属性的值时，我们会重新计算真正的值。

​    之后，我们讨论了 **watch 的实现原理**。它本质上利用了副作用函数重新执行时的可调度性。一个 watch 本身会创建一个 effect，当这个 effect 依赖的响应式数据发生变化时，会执行该 effect 的调度器函数，即 scheduler。这里的 scheduler 可以理解为“回调”，所以我们只需要在 scheduler 中执行用户通过 watch 函数注册的回调函数即可。此外，我们还讲解了立即执行回调的 watch，通过添加新的 immediate 选项来实现，还讨论了如何控制回调函数的执行时机，通过 flush 选项来指定回调函数具体的执行时机，本质上是利用了调用器和异步的微任务队列。

​    最后，我们讨论了**过期的副作用函数**，它会导致竞态问题。为了解决这个问题，Vue.js 为 watch 的回调函数设计了第三个参数，即onInvalidate。它是一个函数，用来注册过期回调。每当 watch 的回调函数执行之前，会优先执行用户通过 onInvalidate 注册的过期回调。这样，用户就有机会在过期回调中将上一次的副作用标记为“过期”，从而解决竞态问题。
