[toc]

### JS 中变量的存储

在百度上搜索「 JavaScript 变量存储」，能看到很多文章，无外乎一个结论：

> 对于原始类型，数据本身是存在栈内，对于对象类型，在栈中存的只是一个堆内地址的引用。

但是，突然想到一个问题：如果说原始类型存在在栈中，那么 JavaScript 中的闭包是如何实现的？（**注：**下面会说不合理之处）

##### 栈

栈是内存中一块用于存储局部变量和函数参数的线性结构，遵循着先进后出的原则。数据只能顺序的入栈，顺序的出栈。当然，栈只是内存中一片连续区域一种形式化的描述，数据入栈和出栈的操作仅仅是栈指针（**注：**栈指针寄存器）在内存地址上的上下移动而已。

栈的特点：轻量，不需要手动管理，函数调时创建，调用结束则消失。

##### 堆

堆可以简单的认为是一大块内存空间，就像一个篮子，你往里面放什么都没关系，但是篮子是私人物品，<font color=FF0000>操作系统并不会管你的篮子里都放了什么，也不会主动去清理你的篮子</font>，<mark>因此在 C 语言中，堆中内容是需要程序员手动清理的，不然就会出现内存溢出的情况</mark>。

既然堆是一个大大的篮子，那么<font color=FF0000>在栈中存储不了的数据（比如一个对象），就会被存储在堆中，**栈中就仅仅保留一个对该数据的引用（也就是该块数据的首地址）**</font>。

**注：**这里补充一张 NJU《计算机系统基础》课程的 PPT 截图：可以看到「栈」和「堆」在内存中的位置，以及他们两者的“数据存储方向”。另外，关于「栈」：「栈」又被称为栈帧 ( stack frame ) 结构，ESP ( Extended Stack Pointer ) 是栈顶指针寄存器（亦即，栈指针），EBP ( Extended Base Pointer ) 是栈底指针寄存器（亦即，帧指针）。

<img src="https://s2.loli.net/2022/04/01/1ESFxit6IGcrKmv.png" alt="image-20220401182634913" style="zoom: 30%;" />

**问题：**既然栈中数据在函数执行结束后就会被销毁，那么 JavaScript 中函数闭包该如何实现？先简单来个闭包：

```js
function count () {
    let num = -1;
    return function () {
        num++;
        return num;
    }
}

let numCount = count();
numCount(); // 0
numCount(); // 1
```

按照结论，num 变量在调用 count 函数时创建，在 return 时从栈中弹出。既然是这样的逻辑，那么调用 numCount 函数如何得出 0 呢？num 在函数 return 时已经在内存中被销毁了啊！

因此，<font color=FF0000>在本例中 JavaScript 的基础类型并不保存在栈中，而应该保存在堆中，供 numCount 函数使用</font>。那么网上大家的结论就是错的了？非也！接下来谈谈我对 JavaScript 变量存储的理解。

既然在 `JavaScript` 中有闭包的问题，抛开栈（`Stack`），仅用堆能否实现变量存储？我们来看一个特殊的例子：

```js
function test () {
    let num = 1;
    let string = 'string';
    let bool = true;
    let obj = {
        attr1: 1,
        attr2: 'string',
        attr3: true,
        attr4: 'other'
    }
    return function log() {
        console.log(num, string, bool, obj);
    }
}
```

伴随着 “test 函数” 的调用，为了保证变量不被销毁，在堆中先生成一个对象就叫 Scope 吧，把变量作为 Scope 的属性给存起来。堆中的数据结构大致如下所示（**注：**注意下图 Scope 和箭头的位置）：

<img src="https://s2.loli.net/2022/04/15/nFN83Elpu7oQXq6.jpg" alt="使用 Scope 保存变量" style="zoom:50%;" />

那么，这样就能解决闭包的问题了吗？当然可以。由于 Scope 对象是存储在堆中，因此返回的 log 函数完全可以拥有 Scope 对象 的访问。下图是该段代码在 Chrome 中的执行效果：

<img src="https://s2.loli.net/2022/04/01/BHMCzFkaQtRsdSJ.jpg" alt="Chrome 中 Scope 的表示" style="zoom:55%;" />

红框部分，与上述一致，同时也反应出了之前提及的问题：<font color=FF0000>例子中 JS 变量并没有存在栈中，而是在堆里，用一个特殊的对象 Scope 保存</font>。（**注：**这里还是很重要，第一次见到 <font color=FF0000>内部属性 \[\[scope]] 中的 Closure 元素，且其中保存着闭包内使用的所有的「自由变量」</font>。另外，这里打印使用的是 console.dir()，自行尝试了下：如果使用 console.log() 只能看到最外层的函数定义）

#### JS 中变量有三种类型

- 局部变量（ [[#局部变量]] ）
- 被捕获的变量（ [[#被捕获变量]] ）
- 全局变量（ [[#全局变量]] ）

##### 局部变量

局部变量很好理解：<font color=FF0000>**在函数中声明**，且 **在函数返回后不会被其他作用域所使用** 的对象</font>。下面代码中的 `local*` 都是局部变量。

```js
function test () {
  let local1 = 1;
  var local2 = 'str';
  const local3 = true;
  let local4 = {a: 1};
  return;
}
```

##### 被捕获变量

<mark>被捕获变量就是 **局部变量的反面**</mark>：<font color=FF0000>在函数中声明，但在函数返回后仍有未执行作用域（函数或是类）使用到该变量，那么该变量就是被捕获变量</font>。下面代码中的 `catch*` 都是「被捕获变量」。**注：**应该可以理解为「自由变量」把？

```js
function test1 () {
  let catch1 = 1;
  var catch2 = 'str';
  const catch3 = true;
  let catch4 = {a: 1};
  return function () {
    console.log(catch1, catch2, catch3, catch4)
  }
}

function test2 () {
  let catch1 = 1;
  let catch2 = 'str';
  let catch3 = true;
  var catch4 = {a: 1};
  return class {
    constructor(){
      console.log(catch1, catch2, catch3, catch4)
    }
  }
}

console.dir(test1())
console.dir(test2())
```

复制代码到 Chrome 即可查看输出对象下的 \[\[Scopes]] 下有对应的 Scope。

##### 全局变量

全局变量就是 global，在 浏览器上为 window 在 node 里为 global。、<font color=FF0000>全局变量会被默认添加到「函数作用域链」的最低端</font>，<font color=FF0000>也就是上述函数中 \[\[Scopes]] 中的最后一个</font>。

全局变量需要特别注意一点：var 和 let / const 的区别：

- **var：**全局的 var 变量其实仅仅是为 global 对象添加了一条属性。毕竟，全局环境下的 var变量 会被挂载到「全局对象」上（即：window / gloabl）。

  **注：**根据 MDN 的说法，在全局作用域中定义的函数，同样会挂载到 全局对象下。示例如下：

  > ```js
  > function greeting() {
  > console.log("Hi!");
  > }
  > 
  > window.greeting(); // It is the same as the normal invoking: greeting();
  > ```
  >
  > **解释：**全局函数 greeting 存储在 window 对象中，像这样：
  >
  > ```js
  > greeting: function greeting() {
  > console.log("Hi!");
  > }
  > ```
  >
  > 摘自：[MDN - 全局对象](https://developer.mozilla.org/zh-CN/docs/Glossary/Global_object)

- **let / const：**全局的 let / const 变量不会修改 window 对象，而是<font color=FF0000>将变量的声明放在了一个特殊的对象下（与 Scope 类似）</font>。示例及在 Chrome Devtools 的运行结果如下：

  ```js
  let testLet = 1
  console.dir(() => {})
  ```

  <img src="https://s2.loli.net/2022/04/01/tWzVYAuJI2f6egD.png" alt="image-20220401220051892" style="zoom:55%;" />

#### 结论

根据上面的说法：<font color=FF0000>**除了局部变量，其他的**</font>（**注：**即，被捕获的变量、全局变量）<font color=FF0000>**全都存在「堆」中**</font>。**注：**即，「局部变量」存在「栈」中

根据变量的数据类型，分为以下两种情况：

- 如果是基础类型，那栈中存的是数据本身
- 如果是对象类型，那栈中存的是堆中对象的引用。

但这是理想情况，再问一个问题：<mark>JS 解析器如何判断一个变量是局部变量呢？</mark> <font color=FF0000>判断出是否被内部函数引用即可</font>（**注：**如果不被使用，则是「局部属性」）。那<mark>如果 JS 解析器并没有判断呢？</mark>那就只能存在堆里。

那么你一定想问，Chrome 的 V8 能否判断出？从结果看应该是可以的。

<img src="https://s2.loli.net/2022/04/01/Z1CcojehHLKIywz.jpg" alt="Chrome 下的局部变量" style="zoom:60%;" />

红框内仅有变量 a，而变量 b 已经消失不见了。**注：**这里 “变量a” 不是局部变量，存在「堆」中；“变量b” 是局部变量，存在「栈」中。

摘自：[JS 变量存储？栈 & 堆？NONONO!](https://juejin.cn/post/6844903997615128583)

#### 《「2021」高频前端面试题汇总之JavaScript篇》中的补充：

**简单数据类型** 直接存储在栈 ( stack ) 中的简单数据段，占据空间小、大小固定，属于被频繁使用数据，所以放入栈中存储。

**引用数据类型** 存储在堆 ( heap ) 中的对象，占据空间大、大小不固定。如果存储在栈中，将会影响程序运行的性能；<font color=FF0000 size=4>**引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址**</font>。<font color=FF0000>当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体</font>。

<font color=FF0000 size=4>**栈区内存由编译器自动分配释放**</font>，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。

<font color=FF0000><font size=4>**堆区内存一般由开发着分配释放**</font>，若开发者不释放，程序结束时可能由垃圾回收机制回收</font>

##### IEEE 754 标准在 JS 数字存储 中的使用

先说背景：为什么 `0.1 + 0.2 !== 0.3` 且 `console.log(0.1 + 0.2)` 的结果为 `0.30000000000000004` ？

0.1 的二进制是 `0.0001100110011001100...`（ 1100 循环），0.2 的二进制是 `0.00110011001100...`（ 1100 循环）；所以加起来的结果是 `0.30000000000000004` 。如何解决：

一个直接的解决方法就是设置一个误差范围，通常称为 “机器精度”。对 JavaScript 来说，这个值通常为 2^-52^ ，在 ES6 中，提供了`Number.EPSILON` 属性，而它的值就是 2^-52^，只要判断 `0.1+0.2-0.3` 是否小于 `Number.EPSILON` ，如果小于，就可以判断为 0.1+0.2 ===0。

```js
function numberepsilon(arg1,arg2){                   
  return Math.abs(arg1 - arg2) < Number.EPSILON;        
}        

console.log(numberepsilon(0.1 + 0.2, 0.3)); // true
```

摘自：[「2021」高频前端面试题汇总之JavaScript篇（上）](https://juejin.cn/post/6940945178899251230)



### 