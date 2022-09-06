[toc]

### JS 变量对象 （词法环境）

**注：**在 ES3 中称为「变量对象」，而在 ES6 中称为 「词法环境」。这里文章是讲述 ES3 的，所以是「变量对象」，上面做了 ES6 中「词法环境」的笔记  [[#词法环境 LexicalEnvironment]]。

**对于每个执行上下文，都有三个重要属性**

- 变量对象 ( Variable object，VO )
- 作用域链 ( Scope chain )
- this

##### 关于「执行上下文」Kuitos 博文中的补充

> ```js
> executionContext：{
>  variable object：vars, functions, arguments,        // 注意，这里的成员
>  scope chain: variable object + all parents scopes   // 注意：all parents scope，重要
>  thisValue: context object
> }
> ```
>
> 摘自：[一道js面试题引发的思考](https://github.com/kuitos/kuitos.github.io/issues/18)

下面重点讲讲创建变量对象的过程

#### 变量对象

变量对象是 与执行上下文相关的 <font color=FF0000>**数据作用域**</font>，<font color=FF0000>存储了在 **上下文中定义的变量和函数声明**</font>。

因为不同执行上下文下的变量对象稍有不同，所以我们来聊聊「全局上下文」下的「变量对象」和「函数上下文」下的「变量对象」。

**补充：**

> 变量对象对于程序而言是不可读的，只有编译器才有权访问变量对象。
>
> 摘自：[一道js面试题引发的思考](https://github.com/kuitos/kuitos.github.io/issues/18)

#### 全局上下文

我们先了解一个概念，叫全局对象。在 [W3School](http://www.w3school.com.cn/jsref/jsref_obj_global.asp) 中也有介绍：

> <mark>「全局对象」是预定义的对象，作为 JavaScript 的 「全局函数」 和 「全局属性」 的占位符</mark>。通过使用全局对象，可以访问所有其他所有预定义的对象、函数和属性。

> 在顶层 JavaScript 代码中，可以用关键字 this 引用全局对象。因为 <font color=FF0000>**全局对象是作用域链的头**</font>，这意味着所有非限定性的变量和函数名都会作为该对象的属性来查询。

> 例如，当 JavaScript 代码引用 parseInt() 函数时，它引用的是全局对象的 parseInt 属性。全局对象是作用域链的头，还意味着在顶层 JavaScript 代码中声明的所有变量都将成为全局对象的属性。

##### 这里简单介绍下「全局对象」

1. 可以通过 this 引用，在客户端 JavaScript 中，全局对象就是 Window 对象。

2. 全局对象是由 Object 构造函数实例化的一个对象。

   ```js
   this instanceof Object === true
   ```

3. 预定义了一堆函数和属性

4. 作为全局变量的宿主

   ```js
   var a = 1
   console.log(a === global.a) // true
   ```

5. 客户端 JavaScript（**注：**这里应该是指“浏览器”） 中，全局对象有 window 属性指向自身

   ```js
   var a = 1
   console.log(this.window.a) // 2
   
   this.window.b = 2;
   console.log(this.b);       // 2
   ```

以上，即：「全局上下文」中的变量对象就是「全局对象」

#### 函数上下文

<font color=FF0000>在**「函数上下文」**中，我们 **用「活动对象」**( activation object, AO ) 来**表示「变量对象」**</font>。

<font color=FF0000>「活动对象」和「变量对象」其实是一个东西</font>，只是<font color=FF0000>**「变量对象」是「规范上的」或者说是「引擎实现上」的，不可在 JavaScript 环境中访问**</font>。<font color=FF0000>**只有到当进入一个「执行上下文」中，这个「执行上下文」的「变量对象」才会被激活**</font>；所以才叫 Activation Object，而 <font color=FF0000>只有被激活的变量对象、也就是活动对象上的各种属性，**才能被访问**</font>。

「活动对象」是在进入「函数上下文」时刻被创建的（**注：**根据下面 [[#关于「函数上下文」Kuitos 博文中的补充]] 的说法，“进入”的意思就是“被调用”、”执行到“），<font color=FF0000>**它（AO）通过函数的 arguments 属性初始化**</font>。arguments 属性值是 Arguments 对象。

##### 关于「函数上下文」Kuitos 博文中的补充

> **<font color=FF0000>当函数被激活</font>，那么一个活动对象 (activation object) 就会被创建并且分配给执行上下文。<font color=FF0000>「活动对象」由特殊对象 arguments 初始化而成</font>。<font color=FF0000>随后，他被当做变量对象 ( variable object ) 用于变量初始化</font>。**
>
> 用代码来说明就是：
>
> ```js
> function a(name, age){
>  var gender = "male";
>  function b(){}
> }
> a(“k”,10);
> ```
>
> <font color=FF0000 size=4>**a 被调用时**</font>（**注：**这里说的很明白了，是 a 被调用之后，创建 AO），<font color=FF0000>**在 a 的执行上下文会创建一个 活动对象 AO，并且被初始化为 AO = [arguments]**</font>。随后 AO 又被当做变量对象 ( variable object ) VO进行变量初始化，此时 VO = [arguments].concat([name,age,gender,b])。
>
> 摘自：[一道js面试题引发的思考](https://github.com/kuitos/kuitos.github.io/issues/18)

#### 执行过程

<font color=FF0000>**「执行上下文」的代码会分成 两个阶段 进行处理：分析 和 执行**</font>，我们也可以叫做：

1. <font color=FF0000>进入执行上下文</font>：[[#进入执行上下文]]
2. <font color=FF0000>代码执行</font>：[[#代码执行]]

##### 进入执行上下文

<font color=FF0000>当进入「执行上下文」时，这时候还没有执行代码</font>，**「变量对象」会包括**：

- <font color=FF0000>函数的**所有形参**</font>（如果是「函数上下文」）。**注：**这个应该对应下面例子中的 有实参的“形参a“

  - 由名称和对应值组成的一个变量对象的属性被创建

  - 没有实参（**注：**可以理解为：调用时没有实参），属性值设为 undefined

- 函数声明

  - <font color=FF0000>由“名称”和“对应值”（**函数对象** ( function-object ) ）组成一个变量对象的属性被创建</font>
  - 如果变量对象已经存在相同名称的属性，则完全替换这个属性（**注：**应该是覆盖？）

- 变量声明。**注：**这里对应下面的变量 b 和 d（ d 是变量声明）

  - 由名称和对应值 ( undefined ) 组成一个「变量对象」的属性被创建（注：如下面的）
  - <font color=FF0000 size=4>**如果「变量名称」跟 “已经声明的形式参数”或函数相同，则变量声明不会干扰已经存在的这类属性**</font>。**注：** 这句话非常重要，解释了「函数提升」和 「var 变量提升」冲突，谁优先（**不是覆盖**）的问题。具体示例参见下面 [[#变量对象的思考题 第二题]]

  **补充：**

  > 这里有一点特殊就是只有 函数声明 ( function declaration ) 会被加入到「变量对象」中，而 <font color=FF0000>**函数表达式 ( function expression )** 则不会</font>。看代码：
  >
  > ```js
  > // 函数声明
  > function a(){}
  > console.log(typeof a); // "function"
  > 
  > // 函数表达式
  > var a = function _a(){};
  > console.log(typeof a); // "function"
  > console.log(typeof _a); // "undefined"
  > ```
  >
  > **函数声明** 的方式下，a 会被加入到变量对象中，故当前作用域能打印出 a。
  > **函数表达式** 情况下，a 作为变量会加入到变量对象中，\_a 作为函数表达式则不会加入，故 a 在当前作用域能被正确找到，\_a 则不会。
  >
  > 另外，关于变量如何初始化，看这里：
  > ![https://github.com/kuitos/kuitos.github.io/blob/assets/images/image2015-3-10%2013-20-41.png](https://s2.loli.net/2022/04/06/7FsKhj6BX9OWpNY.png)
  >
  > 摘自：[一道js面试题引发的思考](https://github.com/kuitos/kuitos.github.io/issues/18)

举个例子：

```js
function foo(a) {
  var b = 2;
  function c() {}
  var d = function() {};

  b = 3;
}

foo(1);
```

在进入执行上下文后，这时候的 AO 是：

```js
AO = {
    arguments: { // arguments
        0: 1,
        length: 1
    },
    a: 1, // 有实参传递，所以有值，不为 undefined
    b: undefined, // 变量声明
    c: reference to function c(){}, // 函数声明
    d: undefined // 函数表达式，是一个变量声明；所以初始化为 undefined
}
```

##### 代码执行

在代码执行阶段，会顺序执行代码，根据代码，修改变量对象的值。

还是上面的例子，当代码执行完后，这时候的 AO 是：

```js
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: 3,
    c: reference to function c(){},
    d: reference to FunctionExpression "d"
}
```

到这里变量对象的创建过程就介绍完了，让我们简洁的总结我们上述所说：

1. 全局上下文的变量对象初始化是全局对象
2. 函数上下文的<font color=FF0000>变量对象初始化只包括 Arguments 对象</font>
3. 在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始的属性值
4. 在代码执行阶段，会再次修改变量对象的属性值

#### 变量对象的思考题

##### 变量对象的思考题 第一题

```js
function foo() {
    console.log(a);
    a = 1;
}
foo(); // ???

function bar() {
    a = 1;
    console.log(a);
}
bar(); // ???
```

第一段会报错：`Uncaught ReferenceError: a is not defined`。第二段会打印：`1`。

这是因为：<font color=FF0000 size=4>**函数中的 “a” 并没有通过 var 关键字声明，所以不会被存放在 AO 中**</font>。

第一段执行 console 的时候， AO 的值是：

```js
AO = {
    arguments: {
        length: 0
    }
}
```

没有 a 的值，然后就会到全局去找，全局也没有，所以会报错。

当第二段执行 console 的时候，全局对象已经被赋予了 a 属性，这时候就可以从全局找到 a 的值，所以会打印 1。

##### 变量对象的思考题 第二题

```js
console.log(foo);
function foo(){
    console.log("foo");
}
var foo = 1;
```

会打印函数，而不是 undefined 。 <font color=FF0000>这是因为在进入执行上下文时，首先会处理函数声明，其次会处理变量声明</font>，<font color=FF0000 size=4>**如果「变量名称」跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性**</font>。另外，上面这句话同样有说过，见 [[#进入执行上下文]]，那里做了一些补充。

摘自：[JavaScript深入之变量对象](https://github.com/mqyqingfeng/Blog/issues/5)

