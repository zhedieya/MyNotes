[toc]

### JS 词法作用域

- <font size=4>**作用域**</font>

  **「作用域」是指：<font color=FF0000>程序源代码中定义变量的区域</font>**。

  <font color=FF0000 size=4>**「作用域」规定了如何查找变量**</font>，也就是确定当前执行代码对变量的访问权限。

  <font color=FF0000 size=4>JavaScript 采用词法作用域 ( lexical scoping)，也就是静态作用域</font>。

- <font size=4>**静态作用域 和 动态作用域**</font>

  因为 JavaScript 采用的是词法作用域，<font color=FF0000 size=4>**函数的作用域 在函数定义的时候就决定了**</font>。

  与词法作用域相对的是 <font color=FF0000>**动态作用域**，函数的作用域是在 **函数调用** 的时候才决定的</font>。动态作用域的语言有：lisp、bash

  **证明 js 是静态作用域的例子：**

  ```js
  var value = 1;
  function foo() { console.log(value); }
  
  function bar() {
      var value = 2;
      foo();
  }
  
  bar(); // 1
  ```

  - **如果是静态作用域：**执行 foo 函数，先从 foo 函数内部查找是否有局部变量 value，如果没有，就根据书写的位置，查找上面一层的代码，也就是 value 等于 1，所以结果会打印 1。
  - **如果是动态作用域：**执行 foo 函数，依然是从 foo 函数内部查找是否有局部变量 value。如果没有，就从调用函数的作用域，也就是 bar 函数内部查找 value 变量，所以结果会打印 2。

  但结果为1

摘自：[JavaScript深入之词法作用域和动态作用域](https://github.com/mqyqingfeng/Blog/issues/3)

##### 博主“冴羽”在issue评论区的引用

此条评论链接：https://github.com/mqyqingfeng/Blog/issues/3#issuecomment-308667350

> 和大多数的现代化编程语言一样，JavaScript 是采用词法作用域的，这就意味着<font color=FF0000>函数的执行依赖于函数定义的时候所产生（而不是函数调用的时候产生的）的变量作用域</font>。<mark style=background:aqua>**为了去实现这种词法作用域**</mark>，<font color=FF0000>JavaScript 函数对象的内部状态不仅包含函数逻辑的代码</font>，除此之外<font color=FF0000>还包含当前作用域链的引用</font>。函数对象可以通过这个作用域链相互关联起来，<mark>如此，函数体内部的变量都可以保存在函数的作用域内，这在计算机的文献中被称之为「闭包」</mark>。
>
> <mark>从技术的角度去将，所有 JavaScript 函数都是「闭包」</mark>：他们都是对象，他们都有一个关联到他们的作用域链。绝大多数函数在调用的时候使用的作用域链和他们在定义的时候的作用域链是相同的，但是这并不影响闭包。当调用函数的时候闭包所指向的作用域链和定义函数时的作用域链不是同一个作用域链的时候，闭包 become interesting。这种 interesting 的事情往往发生在这样的情况下： <mark>当一个函数嵌套了另外的一个函数，外部的函数将内部嵌套的这个函数作为对象返回</mark>。一大批强大的编程技术都利用了这类嵌套的函数闭包，当然，Javascript 也是这样。可能你第一次碰见闭包觉得比较难以理解，但是去明白闭包然后去非常自如的使用它是非常重要的。
>
> 通俗点说，在程序语言范畴内的闭包是指函数把其的变量作用域也包含在这个函数的作用域内，形成一个所谓的“闭包”，这样的话外部的函数就无法去访问内部变量。所以按照第二段所说的，严格意义上所有的函数都是闭包。
>
> 需要注意的是：我们常常所说的闭包指的是让外部函数访问到内部的变量，也就是说，按照一般的做法，是使内部函数返回一个函数，然后操作其中的变量。这样做的话一是可以读取函数内部的变量，二是可以让这些变量的值始终保存在内存中。
>
> 内容的源链接：[rccoder 博客链接](https://github.com/rccoder/blog/issues/3)



### JS 作用域链

> **注意** ⚠️：现在标准已经发生变化，*作用域链* ( scope chain ) 现在在标准中已经变成了 *环境记录* ( Enviornment Record )
>
> 学习自：[鉴定一下网络热门面试题：如何理解闭包的概念？](https://www.bilibili.com/video/BV1b3411w7rX?t=8m56s)

在[《JavaScript深入之执行上下文栈》](https://github.com/mqyqingfeng/Blog/issues/4)中讲到，<font color=FF0000>当 JavaScript 代码执行一段可执行代码 ( executable code ) 时，**会创建对应的执行上下文 ( execution context )**</font>。

**对于每个执行上下文，都有三个重要属性：**

- 变量对象 ( Variable object，VO )。**注：**下面会更详细的说明 VO
- 作用域链 ( Scope chain)
- this

#### 作用域链

在[《JavaScript深入之变量对象》](https://github.com/mqyqingfeng/Blog/issues/5)中讲到，<font color=FF0000>当查找变量的时候，会先从当前上下文的变量对象中查找，**如果没有找到，就会从父级（词法层面上的父级）执行上下文的变量对象中查找**，一直找到全局上下文的变量对象，也就是全局对象</font>。<font size=4>**这样由多个执行上下文的变量对象构成的链表就叫做<font color=FF0000>「作用域链」</font>**</font>。

下面，让我们以一个<font color=FF0000>函数的 <font size=4>**创建**</font> 和 <font size=4>**激活**</font> 两个时期</font>来讲解作用域链是如何创建和变化的（**注：**函数创建是指函数定义，函数激活是指函数被调用？）：

- <font size=4>**函数创建**</font>

  在[《JavaScript深入之词法作用域和动态作用域》](https://github.com/mqyqingfeng/Blog/issues/3)中讲到，<font color=FF0000 size=4>**函数的作用域在函数定义的时候就决定了**</font>。

  这是因为 <font color=FF0000>**函数有一个 内部属性 \[\[scope]]，当函数创建的时候，就会 <font size=4>保存所有父「变量对象」到其中</font>**</font>（**注：**会保存 父变量对象（即：父「执行上下文环境」的 VO ）到 [[scope]] 中，这点要注意！！ 另外，下面有 testScope 函数的 \[\[scope]] 内部属性截图），你 <font color=FF0000 size=4>**可以理解 [[scope]] 就是所有父变量对象的层级链**</font>，但是<font color=FF0000>**注意⚠️：\[\[scope]] 并不代表 完整的作用域链**</font>！

  <img src="https://s2.loli.net/2022/04/05/vT5iDzLo4cqYWPy.png" alt="image-20220405214215002" style="zoom:60%;" />

  **注：**Scope 本来也有 “作用域” 的含义，如下图 Chrome 调试中的 Scope。图片摘自：[现代JS方法 - 在浏览器中调试 - Debugger 命令](https://zh.javascript.info/debugging-chrome#debugger-ming-ling)

  > <img src="https://s2.loli.net/2022/04/05/ATwHPeGcvMKgQnu.png" alt="image-20220405215320848" style="zoom:60%;" />

  **举个例子：**

  ```js
  function foo() { // 注：虽然文中没说，但是要注意：foo 函数是在全局作用域下
      function bar() { ... }
  }
  ```

  函数创建时，各自的 [[scope]] 为：

  ```js
  // 注：之所以这里写成 foo.[[scope]] 是因为 [[scope]] 就是包含在 foo 对象中的；如上面“打印 testScope 函数内容”的图
  foo.[[scope]] = [
    globalContext.VO // variable object
  ];
  bar.[[scope]] = [
    fooContext.AO, // 注：上面有说，AO 是 Activation Object 的缩写
    globalContext.VO
    /* 注：bar 在 foo 的内部，所以 bar 的 [[scope]] 有 fooContext；同时，还有 globalContext，这是我没有想到的；不过，否则的话，[[scope]] 也不会是个数组了。
    另外，开始我以为 [[scope]] 是一个指针，指向上一级的作用域；但现在看来，不是一般认知的指针；或许是个指针数组？*/
  ];
  ```

- <font size=4>**函数激活**</font>
  当函数激活时，进入函数上下文，创建 VO/AO 后，就会将「活动对象」添加到作用链的前端。（**注：**添加到作用链的前端，也就是下面写的操作：[AO].concat(\[\[Scope]])。<font color=FF0000>另外，添加到作用链（一个数组）的最前端，是方便依次查找</font>）

  这时候执行上下文的作用域链，我们命名为 Scope：

  ```js
  Scope = [AO].concat([[Scope]]);
  ```

  至此，作用域链创建完毕。

#### 示例与总结

以下面的例子为例，结合着之前讲的「变量对象」和「执行上下文栈」，我们来总结一下「函数执行上下文」中「作用域链」和「变量对象」的创建过程：

```js
var scope = "global scope";
function checkscope(){
    var scope2 = 'local scope';
    return scope2;
}
checkscope();
```

##### 函数「执行上下文」执行过程如下

1. <font color=FF0000>**checkscope 函数被 <font size=4>创建</font>，保存「作用域链」到 内部属性  \[\[scope]]**</font>（**注：**亦即，父「执行上下文环境」的 VO 保存到 \[\[scope]] ）

   ```js
   checkscope.[[scope]] = [
       globalContext.VO
   ];
   ```

2. <font color=FF0000>执行 checkscope 函数，创建 checkscope 函数「执行上下文」，**checkscope 函数「执行上下文」被压入「执行上下文栈」**</font>

   ```js
   ECStack = [ // 注：这里 ECStack 全称应该是：Execution Context Stack
       checkscopeContext,
       globalContext
   ];
   ```

3. <font color=FF0000 size=4>**checkscope 函数并不立刻执行，开始做准备工作**</font>，<mark>第一步</mark>：<font color=FF0000>复制函数 [[scope]] 属性 <font size=4>**创建作用域链**</font></font>

   ```js
   checkscopeContext = { // 注：这里checkscopeContext 是步骤二中 ECStack 的成员。
       Scope: checkscope.[[scope]],
   }
   ```

4. <mark>第二步</mark>：<font color=FF0000>**用 arguments 创建活动对象**</font>，<font color=FF0000>随后初始化活动对象，加入形参、函数声明、变量声明</font>。（**注：**这里创建 VO 的内容，可以看 [[# JS 变量对象 （词法环境）]] 部分，那里有详细的讲述）

   ```js
   checkscopeContext = {
       AO: { // 这就是创建的 活动对象 Activion object
           arguments: {
               length: 0 // 这里 arguments 是一个实现了 @@iterator 的对象，所以包含有 length 属性。
           },
           scope2: undefined
       }，
       Scope: checkscope.[[scope]],
   }
   ```

5. <mark>第三步</mark>：<font color=FF0000>将活动对象压入 checkscope 作用域链顶端</font>（**注：**即 Scope 数组的开头）

   ```js
   checkscopeContext = {
       AO: {
           arguments: {
               length: 0
           },
           scope2: undefined
       },
       Scope: [AO, [[Scope]]] // 这里，AO 被压入 checkscope作用域顶端
   }
   ```

6. <font color=FF0000 size=4>准备工作做完，开始执行函数</font>，随着函数的执行，修改 AO 的属性值

   ```js
   checkscopeContext = {
       AO: {
           arguments: {
               length: 0
           },
           scope2: 'local scope' // scope2 的值被修改
       },
       Scope: [AO, [[Scope]]]
   }
   ```

7. 查找到 scope2 的值，<font color=FF0000>返回后函数执行完毕，<font size=4>**函数上下文从执行上下文栈中弹出**</font></font>（注：即 checkscopeContext 被弹出、销毁）

   ```js
   ECStack = [
       globalContext
   ];
   ```

摘自：[JavaScript深入之作用域链](https://github.com/mqyqingfeng/Blog/issues/6)

