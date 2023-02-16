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

- 当副作用函数执行时会出发obj.text的**读取**操作
- 当修改obj.text时，会出发字段obj.text的**设置**操作

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

选择`WeakMap`来存储是因为WeakMap对key是弱引用，若用户侧对代码对target没有任何引用，target会被垃圾回收器回收。

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

