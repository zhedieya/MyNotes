[toc]

---



# vue

### 1. MVVM（Model-View-ViewModel） 模式和MVC 模式分别是什么？有什么区别？

- **MVC:** **所有通信都是单向的**
  - 视图（View）：用户界面。（传送指令到 Controller）
  - 控制器（Controller）：业务逻辑（完成业务逻辑后，要求 Model 改变状态）
  - 模型（Model）：数据保存（将新的数据发送到 View，用户得到反馈）
    ![image](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015020105.png?ynotemdtimestamp=1652084783499)
- **MVVM：前后端分离：各部分之间的通信，都是双向的，View 与 Model 不发生联系，都通过ViewModel传递**

![image](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015020110.png?ynotemdtimestamp=1652084783499)

**ViewModel把view和model关联起来，ViewModel负责把Model的数据同步到view显出来，还负责把view修改同步到Model。**

[答案详情](https://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)

### 2. 【重点】vue2的响应式原理的实现?

> Vue是一个典型的MVVM框架，模型（Model）只是普通的JavaScript对象，修改它则视图（View）会自动更新。这种设计让状态管理变得非常简单而直观。那么Vue是如何把模型和视图建立起关联的呢？

![image](https://image-static.segmentfault.com/132/184/132184689-57b310ea1804f_articlex?ynotemdtimestamp=1652084783499)

**依赖收集**

视图中会用到`data`中某`key`，这称为依赖。同⼀个`key`可能出现多次，每次都需要收集出来用⼀个`Watcher`来维护它们，此过程称为依赖收集；多个`Watcher`需要⼀个`Dep`来管理，需要更新时由`Dep`统⼀通知。

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20220729233111773.png" alt="image-20220729233111773" style="zoom:50%;" />

**vue通过三大模块来实现的**：

1. **Observer**: 能对数据对象的所有属性进行监听，如有订阅可拿到最新值并通知订阅者
2. **Compiler**:对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数
3. **Watcher**: 操作Observer和Compile的桥梁，能够订阅并收到每个属性变化的的通知，执行指令绑定的相应回调函数，从而更新视图

**一句话总结vue底层逻辑：创建vue实例后，遍历data选项的数据，通过object.defineProperty为属性添加getter和setter对数据进行劫持，劫持数据时会为属性创建 dep 用来收集watcher，当数据更新时通过dep.notify()通知watcher,派发更新，并且触发compiler中绑定的回调，渲染视图**



**长话短说：劫持数据，创建dep通知watcher，触发回调，更新数据，渲染视图**

一个属性对象多个dep

一个dep对象多个watcher

一个watcher对应多个dep

dep和watcher是多对多关系

[答案详情1](https://juejin.cn/post/6844904031983239181)
[答案详情2](https://segmentfault.com/a/1190000006599500?utm_source=sf-related)

### 【重点】v-model和v-text的区别？

- v-model通常用于表单组件的绑定，表单组件的双向绑定
- v-text用于操作纯文本，单向绑定，数据变化->插值跟着变化，但插值变化不会影响数据对象的值

### 3. 【重点】Vue的生命周期方法有哪些？一般在哪一步发送请求及原因?

[Vue生命周期](/Users/owsl/Desktop/works/MyNotes/Vue/1.基础/07.Vue基础-Vue生命周期.md)

答题思路：什么时候执行？实例内部发生了什么变化？能进行什么业务操作？

- `beforeCreate` 在实例初始化之后，数据观测(data observe)和watcher配置之前被调用；在此可加载loading事件
- `created` 实例已经创建完成之后被调用。在这一步实例已经完成数据观测(data observe)和watcher事件回调，但实例还未挂载到DOM上；可在此结束beforeCreate中的loading事件
- `beforeMount` 在挂载开始之前被调用：找到对应的template模板，并编译成render函数。
- `mounted` 实例挂载到DOM上了（可以通过DOM api回去到节点，`ref`属性可以访问了）
- `beforeUpdate` 虚拟DOM重新渲染和打补丁之前调用
- `updated` 虚拟 DOM 重新渲染和打补丁之后调用，组件DOM已经更新，可执行依赖于DOM的操作
- `beforeDestroy` 实例销毁之前调用，这一步实例仍然完全可用。
- `destroyed` 实例销毁之后调用，调用后实例绑定的所有东西都会解绑，所有事件监听会被销毁，所有的子实例也会被销毁
- `keep-alive` (activated 和 deactivated)

#### 组件生命周期

生命周期（父子组件） 父组件beforeCreate --> 父组件created --> 父组件beforeMount --> 子组件beforeCreate --> 子组件created --> 子组件beforeMount --> 子组件 mounted --> 父组件mounted -->父组件beforeUpdate -->子组件beforeDestroy--> 子组件destroyed --> 父组件updated

**加载渲染过程** 父beforeCreate->父created->父beforeMount->子beforeCreate->子created->子beforeMount->子mounted->父mounted

**挂载阶段** 父created->子created->子mounted->父mounted

**父组件更新阶段** 父beforeUpdate->父updated

**子组件更新阶段** 父beforeUpdate->子beforeUpdate->子updated->父updated

**销毁阶段** 父beforeDestroy->子beforeDestroy->子destroyed->父destroyed

[答案详情](https://juejin.cn/post/6844903602356502542)

### 4. 【重点】谈谈对vue组件化的理解

- **高内聚低耦合，单向流数据**

- **提高开发效率，和复用性**

- **降低更新范畴，只重新渲染变化的组件，可以提高性能**

  比如说当某个组件的数据改变时，它只会重新渲染数据改变的那个组件的dom，不会重现渲染整个的Dom

### 5. 【重点】v-for为什么要加key，能用index作为key吗？

- **可以复用dom节点，提升性能**
- **用index作为key和不加key是一样的，都采用“就地复用”的策略**
- diff算法默认使用 “就地复用”的策略
- “就地复用”原则只适用于不依赖子组件状态或临时dom状态（例如：表单输入）

[答案详情地址](https://www.cnblogs.com/youhong/p/11327062.html)

### 6. 【重点】vue组件之间的通信？

[组件间通信](/Users/owsl/Desktop/works/MyNotes/收集的好文&面经/面试官系列/vue/communication.md)

怎么回答？**什么场景用什么方法？项目中有哪些地方用到了？**

- 父子关系的组件数据传递选择 `props` 与 `$emit`进行传递，也可选择`ref`
- 兄弟关系的组件数据传递可选择`$bus`，其次可以选择`$parent`进行传递
- 祖先与后代组件数据传递可选择`attrs`与`listeners`或者 `Provide`与 `Inject`
- 复杂关系的组件数据传递可以通过`vuex`存放共享的变量

方法：

- 方法一：

  ```
  props/$emit
  ```

  - 父传子：子组件通过`props`访问父组件的值
  - 子传父：子组件通过`$emit`自定义事件向父组件发送数据

- 方法二、三：`$parent/$children`与`ref`

> `$parent/$children`与`ref`这两种都是直接得到组件实例，使用后可以直接调用组件的方法或访问数据

> **但这两种方法无法在跨级或兄弟间通信，不过可以借助vuex来实现**

- 方法三：`$attrs` 和 `$listeners` A->B->C。Vue 2.4 开始提供了 `$attrs` 和 `$listeners` 来解决这个问题
- 方法四：通过 `$emit`和`$on`实现`eventBus`进行数据传递
- 方法五：父组件中通过 `provide` 来提供变量，然后在子组件中通过 `inject` 来注入变量。
- 方法六：通过`vuex`状态管理实现组件之间的通信
- 方法七：甚至可通过 `localStorage/sessionStorage`

[答案详情](https://juejin.cn/post/6844903887162310669#heading-21)

### 7. 【重点】对template模板编译的理解

问题核心：如何将template转换成render函数 ?

1. 将template模板转换成 ast 语法树 - parserHTML
2. 对ast语法树做标记静态节点，优化ast - markUp
3. 将ast语法树转化为可执行的代码render - codeGen

简而言之，就是先将template转化成AST树，再将ast树转化成render函数

```js
export const createCompiler = createCompilerCreator(function
  baseCompile(
    template: string,
    options: CompilerOptions
  ): CompiledResult {
  const ast = parse(template.trim(), options) // 1.解析ast语法树
  if (options.optimize !== false) {
    optimize(ast, options)           // 2.对ast树进行标记,标记
    静态节点
  }
  const code = generate(ast, options)     // 3.生成代码
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
})
```

### 8. 【重点】computed的实现原理

每个computed计算属性都会对应一个 Watcher实例，通过watcher来实现的

### 9. 【重点】computed和watch的区别？

> computed：多个变量影响一个变量(**多对一**)  watch：一个变量影响多个变量(**一对多**) 
>
> 通俗来讲，既能用 computed 实现又可以用 watch 监听来实现的功能，推荐用 computed

- computed和watch都是基于watcher来实现的
- 如果一个数据依赖受其他数据影响，就用computed，且**computed有缓存**，依赖不变computed不会进行重新计算操作，可减少开销，提高性能
- 当需要在数据变化时执行异步或开销较大的操作时，就使用watch

### 10. 【理解即可】Vue.set()方法是如何实现的？

- 给对象和数组本身都添加的`def`属性

- 当给对象新增属性的时候，会触发依赖watcher去更新对象

- 当改变数组的索引时，会重写数组splice()方法更新数组

```js
export function set(target: Array<any> | Object, key: any, val:
  any): any {
  // 1.是开发环境 target 没定义或者是基础类型则报错
  if (process.env.NODE_ENV !== 'production' &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn(`Cannot set reactive property on undefined, null, or
  primitive value: ${(target: any)}`)
  }
  // 2.如果是数组 Vue.set(array,1,100); 调用我们重写的splice方法 (这样可以
  更新视图)
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key)
    target.splice(key, 1, val)
    return val
  }
```

### 11. 【重点】Vue组件data为什么必须是个函数？

避免组件被复用时，数据存在引用关系

1.一个组件被复用多次的话，也就会创建多个实例。本质上，这些实例用的都是同一个构造函数。 

2.如果data是对象的话，对象属于引用类型，会影响到所有的实例。所以为了保证组件不同的实例之间data不冲突，data必须是一个函数。

### 12. 【重点】nextTick在哪里使用？原理是？

首先得知道Vue更新DOM的机制，Vue官网对数据操作是这么描述的：

> 可能你还没有注意到，Vue 在更新 DOM 时是异步执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 Promise.then、MutationObserver 和 setImmediate，如果执行环境不支持，则会采用 setTimeout(fn, 0) 代替。

也就是说我们在设置`this.msg = 'some thing'`的时候，Vue并没有马上去更新DOM数据，而是将这个操作放进一个队列中；如果我们重复执行的话，队列还会进行去重操作；等待**同一事件循环中**的所有数据变化完成之后，会将队列中的事件拿出来处理。

这样做主要是为了提升性能，因为如果在主线程中更新DOM，循环100次就要更新100次DOM；但是如果等事件循环完成之后更新DOM，只需要更新1次。

为了在数据更新操作之后操作DOM，我们可以在数据变化之后立即使用`Vue.nextTick(callback)`；这样**回调函数**会在DOM更新完成后被调用，就可以拿到最新的DOM元素了。

官方对nextTick解释是：**将回调延迟到下次DOM更新循环之后执行。在修改数据之后立即使用它，然后等待DOM更新。**

> 一句话就可以把`$nextTick`这个东西讲明白：就是你放在`$nextTick `当中的操作不会立即执行，而是等数据更新、DOM更新完成之后再执行，这样我们拿到的肯定就是最新的了。

- 可用来获取更新后的`Dom`
- Vue中数据更新是异步的,可以保证`nextTick`里面的回调函数在`Dom`重新渲染之后执行

[使用场景例子](https://juejin.cn/post/6844904197515640839)

### 13. 【重点】$nextTick的原理是什么？

> 每次event loop的最后，会有一个UI render步骤，也就是更新DOM

**原理：在dom更新之后的下一次的event loop 事件循环中执行nexttick里面的回调函数的异步任务，vue会采用Promise.then()、MutationObserve、setTimeout来执行nexttickhandler，依次测试浏览器支持的方法，如果前两个都不支持就用最后一个setTimeout来执行。**

- Vue.nextTick([callback, context])是全局的，
- 使用vm.$nextTick([callback])时的回调会自动绑定到调用它的实例上。

```js
  return function queueNextTick (cb, ctx) {
    var _resolve;
    callbacks.push(function () {
    //看这里，其实是可以给cb指定一个对象环境，来改变cb中this的指向
      if (cb) { cb.call(ctx); }
      if (_resolve) { _resolve(ctx); }
    });
    if (!pending) {
      pending = true;
      timerFunc();
    }
    if (!cb && typeof Promise !== 'undefined') {
      return new Promise(function (resolve) {
        _resolve = resolve;
      })
    }
  }
```

**源码中使用的是`if (cb) { cb.call(ctx) }`,用.call()改变this指向， 所以不能使用箭头函数，箭头函数的this是固定的，是不可用apply,call,bind来改变的。**

[参考链接1](https://www.jianshu.com/p/0f833d3881b9?from=groupmessage)

[参考链接2](https://www.jianshu.com/p/7f9495b1c8ab)

[参考链接3](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/281)

### 14.为什么v-for和v-if不建议用在一起

因为v-for的优先级高于v-if，若同时使用，每次v-for都会执行v-if,造成性能浪费。

如果避免出现这种情况，则在外层嵌套`template`（页面渲染不生成`dom`节点），在这一层进行v-if判断，然后在内部进行v-for循环

```js
<template v-if="isShow">
    <p v-for="item in items">
</template>
```

如果条件出现在循环内部，可通过计算属性`computed`提前过滤掉那些不需要显示的项

```js
computed: {
    items: function() {
      return this.list.filter(function (item) {
        return item.isShow
      })
    }
}
```



### 15.v-if和v-show区别

- 控制手段不同
- 编译过程不同
- 编译条件不同

控制手段：`v-show`隐藏则是为该元素添加`css--display:none`，`dom`元素依旧还在。`v-if`显示隐藏是将`dom`元素整个添加或删除

编译过程：`v-if`切换有一个局部编译/卸载的过程，切换过程中合适地销毁和重建内部的事件监听和子组件；`v-show`只是简单的基于css切换

编译条件：`v-if`是真正的条件渲染，它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。只有渲染条件为假时，并不做操作，直到为真才渲染

- `v-show` 由`false`变为`true`的时候不会触发组件的生命周期
- `v-if`由`false`变为`true`的时候，触发组件的`beforeCreate`、`create`、`beforeMount`、`mounted`钩子，由`true`变为`false`的时候触发组件的`beforeDestory`、`destoryed`方法

性能消耗：`v-if`有更高的切换消耗；`v-show`有更高的初始渲染消耗；



### 16.mixin

官方定义

> `mixin`（混入），提供了一种非常灵活的方式，来分发 `Vue` 组件中的可复用功能。

本质其实就是一个`js`对象，它可以包含我们组件中任意功能选项，如`data`、`components`、`methods`、`created`、`computed`等等

我们只要将共用的功能以对象的方式传入 `mixins`选项中，当组件使用 `mixins`对象时所有`mixins`对象的选项都将被混入该组件本身的选项中来

在`Vue`中我们可以**局部混入**跟**全局混入**

**局部混入**

定义一个`mixin`对象，有组件`options`的`data`、`methods`属性

```js
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}
```

组件通过`mixins`属性调用`mixin`对象

```js
Vue.component('componentA',{
  mixins: [myMixin]
})
```

该组件在使用的时候，混合了`mixin`里面的方法，在自动执行`create`生命钩子，执行`hello`方法

**全局混入**

通过`Vue.mixin()`进行全局的混入

```js
Vue.mixin({
  created: function () {
      console.log("全局混入")
    }
})
```

使用全局混入需要特别注意，因为它会影响到每一个组件实例（包括第三方组件）

PS：全局混入常用于插件的编写

> 通常使用mixin将功能相似的代码提取出来

------

# Vuex

### 1. vuex的工作流程

组件应用state
组件调用dispatch发布到action里面，在action里面可以进行异步网络请求，然后commit提交到mutation里面，在mutation里面修改state

需要注意的是：网络请求一般在action里面进行，不要在action里面修改state，因为devtool只能跟踪mutations里面的修改记录

[答案详情](https://segmentfault.com/a/1190000034866486)

[思维导图](https://naotu.baidu.com/file/0bc66aa151f88d766c30f4f6e2b86d94)

### 2. 【重点】vuex的5大核心

**分别是state、getters、mutations、actions、modules**

- **state是状态、**
- **getters：相当于vue中的计算属性computed**

```js
getters: {
  total: state => {
    return state.price * state.number
  },
    discountTotal: (state, getters) => {
      return state.discount * getters.total
    }
},
```

- **mutations：改变Vuex中的状态的唯一途径就是显式地提交 (commit) mutation。**
- **actions：进行异步操作，比如网络请求**
- **modules：store对象变得臃肿就需要分模块**

### 3. 【重点】为什么要在computed获取vuex中的状态state，而不在data中？

**因为data中的内容只会在create钩子触发前初始化一次**。具体来说就是data中设置`count: this.$store.state.count` 则`count`的值是created钩子执行前`this.$store.state.count`的值，赋值之后属性的值就是纯粹的字面量，之后`this.$store.state.count`如何变化均影响不到count的取值。就如同下面这段代码:

```js
var b = 1;
var a = b;
b = 2;
```

而 computed 则是通过【依赖追踪】实现的，计算属性在它的相关依赖发生改变时会重新求值（可参考[vue官方教程对计算属性的描述](https://cn.vuejs.org/v2/guide/computed.html)），所以可以使用 computed 去引用 Vuex 状态变量，从而使得依赖追踪生效

[答案地址](https://blog.csdn.net/wopelo/article/details/80244049)

------

# Vue-Router

[Vue-Route思维导图](https://naotu.baidu.com/file/e3d2a293b235b23f2497367170675f29)

[问题地址](https://juejin.cn/post/6844903945530245133)

[参考问题地址](https://juejin.cn/post/6844903961745440775#heading-13)

### 16.【重点】切换路由时，需要保存草稿的功能，怎么实现呢？

**可以根据使用方法结合实际项目说**

```html
<keep-alive :include="include">
    <router-view></router-view>
</keep-alive>
```

其中include可以是个数组，数组内容为路由的name选项的值。

### 17.【重点】怎么配置404页面？

给 path 配置通配符，然后用 redirect 重定向。示例代码：

```js
const router = new VueRouter({
    routes: [
        {
           path:'*',redirect: {path:'/'}
        }
    ]
})
```

### 18.【重要】在什么场景下会用到嵌套路由？（结合项目）

**例如我做的这个后台管理系统，顶部栏和左侧菜单栏是全局通用的，把它当作父路由，右下侧的页面的内容部分放在子路由里**

[答案地址](https://juejin.cn/post/6844903961745440775#heading-15)

### 19.【重点】什么是命名视图<router-view name=""></router-view，举个例子说明一下？（结合项目说）

**例如在我这个后台管理系统对的项目中，我们想同级展示多个视图，而不是嵌套展示。例如项目首页，有头部导航，侧边栏导航、主内容区域。头部导航、侧边栏导航我们不想用组件方式引入，想用`<router-view name=""></router-view>`视图方式展示。那么这个首页上，就有三个视图，头部导航视图，侧边栏导航视图、主内容区域视图同级展示。**

[答案地址](https://juejin.cn/post/6844903961745440775#heading-16)

### 20.【重点】Vue-Router有哪些组件？

- **导航组件**`<router-link to="" tag="" active-class=""></router-link>`
- 渲染路径匹配到的**视图组件**`<router-view></router-view>`
- **vue的内置组件**：`<keep-alive include="" exclude=""></keep-alive>`

### 21.【重点】active-class是哪个组件的属性？

时`<router-link/>`组件的属性，设置链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 linkActiveClass 来全局配置。

### 22.【重点】怎么定义Vue-Router的动态路由？怎么获取传过来的值？

以冒号加参数的形式

```
path: 'user/:id'
```

在组件内通过路由对象route访问,例如
`this.$route.params.id`

### 23.【重点】Vue-Router有哪几种导航钩子（Vue-Router有哪些导航守卫方法）？

- 全局的 `beforeEach`
- 全局的 `beforeResolve` 导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后被调用
- 全局的 `afterEach`
- 组件内的 `beforeRouterEnter`
- 组件内的 `beforeRouterUpdate`
- 组件内的 `beforeRouterLeave`

### 24.【了解即可】Vue-Router完整导航解析流程是什么？

离开的组件的相关方法->全局beforeEach守卫->重用的组件的相关方法->路由配置里面的beforeEnter->（解析路由）调用被激活的组件的相关方法(beforeRouteEnter)->全局的beforeResolve->导航被确认->afterEach钩子->触发dom更新->调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。

- `beforeRouterLeave`
- `beforeEach`
- `beforeRouterUpdate`
- `beforeEnter`
- `beforeRouterEnter`
- `beforeResole` 在导航被确认之前，同时所有组件内守卫和异步路由组件被解析之后
- 导航被确认
- `afterEach`
- 触发 DOM 更新
- `beforeRouterEnter`中传给next的回调函数，创建好的组件实例会作为回调函数的参数传入。

[答案地址](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#组件内的守卫)

### 25.【重点】$route和$router的区别是什么？

- router是VueRouter实例，是一个全局路由对象，通过它可以调用路由跳转方法来跳转页面、钩子函数等等；
- route是当前路由信息的对象，它是局部对象，每个路由都会有一个route对象，它里面包是当前路由对象信息，有path、params、query、name等信息。

### 26.【重点】Vue-Router相应路由参数的变化

- 通过watch检测
- 通过组件内导航钩子

### 27.【重点】Vue-Router传参方式

需要知道：分别由哪几种方式？有什么区别？

- 通过

  ```
  Params
  ```
  
  - 只能使用 name 不能使用 path 路径
  - 参数不会显示在路径上
  - 浏览器强制刷新，参数会被清楚

```JavaScript
this.$router.push{
    name: Home,
    params: {
        number: 1,
        code: 99
    }
}
```

- 通过 

  ```
  query
  ```
  
  - 可以使用 name 或者 path 路径
- 参数会显示在路径上，浏览器刷新不会被清空

### 28.【重点】Vue-Router的两种模式

- ```
  hash
  ```

  模式：

  - 原理是`onhashchange`事件，`hashchange` 只能改变 `#` 后面的代码片段。可以在window对象上监听这个事件

```javascript
window.onhashchange = function(event){
  console.log(event.oldURL, event.newURL)
  let hash = location.hash.slice(1)
}
```

- `history`模式
  - 利用了HTML5 中新增的`pushState()`和`replaceState()`方法
  - 需要后台配置支持。如果刷新时，服务器没有响应的资源，会刷出404，
- `abstract`
  - 支持所有 `JavaScript` 运行环境

### 29.【重点】Vue-Router实现路由懒加载（动态加载）

[应用场景](https://blog.csdn.net/u014678583/article/details/107387891)

**把导入路由写成方法的形式，然后在配置路由映射的时候把component对应导入路由的方法，当路由被访问时才执行导入路由的方法**

例子：

```js
const router = new VueRouter({
  routes: [
    {
      path: '/home',
      name: 'Home'，
      component:() => import('../views/home')
		}
  ]
})
```

### 30.【重点】路由之间是怎么跳转的？有哪些方式？

- 通过使用内置组件：`<router-link :to="/home">`来跳转
- 通过调用router实例的push方法或者replace方法

### 31.【重点】路由跳转方法 push 和 repalce 的区别?

- 使用replace不会向history添加新记录，而是替换当天页面的history记录

### 32.【重点】后台管理系统项目中怎么获取菜单栏的？菜单栏的路由地址怎么实现的？

> 1.后台同学返回一个json格式的路由表，我用easymock造了一段:动态路由表，大家可参考;
>
> 2.因为后端同学传回来的都是字符串格式的，但是前端这里需要的是一个组件对象啊，写个方法遍历一下，将字符串转换为组件对象;
>
> 3.利用vue-router的beforeEach、addRoutes、localStorage来配合上边两步实现效果;
>
> 4.左侧菜单栏根据拿到转换好的路由列表进行展示;
> 大体步骤：**拦截路由->后台取到路由->保存路由到localStorage(用户登录进来只会从后台取一次，其余都从本地取,所以用户，只有退出在登录路由才会更新)**

[答案地址](https://segmentfault.com/a/1190000015419713)

### 33. 【重要】能在vue-router钩子beforeRouteEnter函数里面获取组件实例this吗？

不能

### 34. 【重要】谈谈vue2和vue3的区别（vue3有哪些有点？）

- vue3中采用了composition Api
- vue2 用 es6 的是 Object.defineProperty 监听对象 ；vue3采用 proxy 代理 监听对象，
- vue3 TypeScript 有了很好的支持

`Object.defineProperty` 的缺点：

- 监听不到数组的变化，vue2是通过重写数组的push、pop、shift、unshift、splice、sort、reverse方法来实现数组的监听
- 必须遍历对象的每个属性（Object.defineProperty多数要配合Object.keys使用）
- 必须深层遍历嵌套的对象

`proxy` 的优点：

- 针对对象而不是某个属性，省略了遍历每个属性的过程，提高了性能
- 支持对象嵌套：get里面递归调用proxy并返回

### 【重要】单页面应用程序（SPA）的优缺点

定义：只在一个Web页面初始化时加载相应的HTML、js、css。避免了页面的重新加载。

- 优点：
  - 单页面内容的改变不需要重新加载整个页面，可以通过ajax异步获取数据
  - 减轻服务器压力，后端不需要管模板渲染
- 缺点：
  - 不利于SEO,SEO 本质是一个服务器向另一个服务器发起请求，解析请求内容