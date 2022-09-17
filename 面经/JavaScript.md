[toc]



#### script标签设置为 async 和 defer 的区别

`script` ：会阻碍 HTML 解析，只有下载好并执行完脚本才会继续解析 HTML。

`async script` ：解析 HTML 过程中进行脚本的异步下载，下载成功立马执行，有可能会阻断 HTML 的解析。

`defer script`：完全不会阻碍 HTML 的解析，解析完成之后再按照顺序执行脚本。

<img src="/Users/owsl/Library/Application Support/typora-user-images/image-20220413142542156.png" alt="image-20220413142542156" style="zoom:30%;" />



#### 为什么js是单线程的

```javascript
- 因为多线程的复杂性，多线程操作需要加锁，编码的复杂性会增高。
- 如果同时操作 DOM ，在多线程不加锁的情况下，最终会导致 DOM 渲染的结果不可预期。
```



#### 原型与原型链

[轻松理解JS 原型原型链](https://juejin.cn/post/6844903989088092174)

```css
js分为函数对象和普通对象，每个对象都有__proto__属性，但是只有函数对象才有prototype属性
原型：每一个 JavaScript 对象（null 除外）在创建的时候就会与之关联另一个对象，这个对象就是我们所说的原型，每一个对象都会从原型"继承"属性，其实就是 prototype 对象。
原型链：由相互关联的原型组成的链状结构就是原型链。

实例对象可通过_proto_属性指向该对象的原型，对象正是依靠这个属性连结在一起形成了原型链，作为一个对象，当你访问其中的一个属性或方法的时候，如果这个对象中(构造函数)没有这个方法或属性，就通过_proto_属性到他的原型对象里找，如果原型对象中有，则使用，因为原型对象本身是普通对象，、所以如果没有则去原型对象的原型中寻找,直到找到Object对象的原型，如果在Object原型中依然没有找到，则返回null
```

<img src="/Users/owsl/Library/Application Support/typora-user-images/image-20220324105159788.png" alt="image-20220324105159788" style="zoom:40%;" align="left"/>



<img src="/Users/owsl/Library/Application Support/typora-user-images/image-20220421211601088.png" alt="image-20220421211601088" style="zoom:50%;" />

#### 继承的多种方式和特点

* 原型链继承

  ```javascript
  function Parent(){
      this.names=['aa','bb'];
  }
  function Child(){
  }
  
  Child.prototype=new Parent();
  Child.prototype.constructor=Child;
  
  var child1=new Child();
  child1.names.push('cc');
  console.log(child1.names);  //['aa','bb','cc']
  
  //缺点:引用类型的属性被所有实例共享
  var child2=new Child();
  console.log(child2.names);
  ```

* 使用构造函数  借助`call`调用Parent函数

  ```javascript
  function Parent(){
      this.name = 'parent1';
  }
  
  Parent.prototype.getName = function () {
      return this.name;
  }
  
  function Child(){
      Parent.call(this);
      this.type = 'child'
  }
  
  let child = new Child();
  console.log(child);  // Child { name: 'parent1', type: 'child' }
  console.log(child.getName());  // 会报错
  
  //相比第一种原型链继承方式，父类的引用属性不会被共享，优化了第一种继承方式的弊端，但是只能继承父类的实例属性和方法，不能继承原型属性或者方法
  ```

* 组合继承

  ```javascript
  //结合了原型链继承和构造函数继承
  function Parent(name){
      this.name = name;
      this.colors = ['red', 'blue', 'green'];
  }
  
  Parent.prototype.getName=function(){
      console.log(this.name);
  }
  
  function Child(name,age){
      Parent.call(this,name);  //第一次调用
      this.age=age;
  }
  
  //第二次调用
  Child.prototype=new Parent();
  // 手动挂上构造器，指向自己的构造函数
  Child.prototype.constructor=Child;
  
  var child1 = new Child('kevin', '18');
  child1.colors.push('black');
  console.log(child1.name); // kevin
  console.log(child1.age); // 18
  console.log(child1.colors); // ["red", "blue", "green", "black"]
  
  var child2 = new Child('daisy', '20');
  console.log(child2.name); // daisy
  console.log(child2.age); // 20
  console.log(child2.colors); // ["red", "blue", "green"]
  
  //优化了原型链继承和构造函数继承，但Parent函数调用了两次，造成了多调用一次的性能开销
  
  ```
  

- 寄生组合继承(最常用)

  ```javascript
  function clone (parent, child) {
      // 这里改用 Object.create()就可以减少组合继承中多进行一次构造的过程
      child.prototype = Object.create(parent.prototype);
      child.prototype.constructor = child;
  }
  
  function Parent6() {
      this.name = 'parent6';
      this.play = [1, 2, 3];
  }
  Parent6.prototype.getName = function () {
      return this.name;
  }
  function Child6() {
      Parent6.call(this);
      this.friends = 'child5';
  }
  
  //继承
  clone(Parent6, Child6);
  
  Child6.prototype.getFriends = function () {
      return this.friends;
  }
  
  let person6 = new Child6(); 
  console.log(person6); //{friends:"child5",name:"child5",play:[1,2,3],__proto__:Parent6}
  console.log(person6.getName()); // parent6
  console.log(person6.getFriends()); // child5
  ```

  

- es6 extends关键字继承

  ```javascript
  class Person {
    constructor(name) {
      this.name = name
    }
    // 原型方法
    // 即 Person.prototype.getName = function() { }
    // 下面可以简写为 getName() {...}
    getName = function () {
      console.log('Person:', this.name)
    }
  }
  class Gamer extends Person {
    constructor(name, age) {
      //super 关键字用于访问和调用对象父类上的函数。可以调用父类的构造函数，也可以调用父类的普通函数
      // 子类中存在构造函数，则需要在使用“this”之前首先调用 super()。
      super(name)
      this.age = age
    }
  }
  const asuna = new Gamer('Asuna', 20)
  asuna.getName() // 成功访问到父类的方法
  ```

<img src="/Users/owsl/Desktop/works/MyNotes/图/继承.png" alt="继承" style="zoom:50%;" align="left"/>



#### 事件的传播

事件的传播

关于事件的传播网景公司和微软公司有不同的理解
- 微软公司认为事件应该是由内向外传播，也就是当事件触发时，应该先触发当前元素上的事件，然后再向当前元素的祖先元素上传播，也就说事件应该在冒泡阶段执行。
  
- 网景公司认为事件应该是由外向内传播的，也就是当前事件触发时，应该先触发当前元素的最外层的祖先元素的事件，然后在向内传播给后代元素
  
- W3C综合了两个公司的方案，将事件传播分成了三个阶段
  
	在**标准事件模型**中，一次事件有三个过程
  
	**1.捕获阶段**

    在捕获阶段时从最外层的祖先元素，向目标元素进行事件的捕获，但是默认此时不会触发事件
  
  **2.目标阶段**

​     事件捕获到目标元素，捕获结束开始在目标元素上触发事件

   **3.冒泡阶段**

​     事件从目标元素向他的祖先元素传递，依次触发祖先元素上的事件，如果希望在捕获阶段就触发事件，可以将addEventListener()的第三个参数设置为true，一 

​     般情况下我们不会希望在捕获阶段触发事件，所以这个参数一般都是false
​	

#### 数据类型的判断

##### typeof

> 能判断所有**值类型，函数**。不可对 **null、对象、数组**进行精确判断，因为都返回 `object`

```js
console.log(typeof undefined); // undefined
console.log(typeof 2); // number
console.log(typeof true); // boolean
console.log(typeof "str"); // string
console.log(typeof Symbol("foo")); // symbol
console.log(typeof 2172141653n); // bigint
console.log(typeof function () {}); // function
// 不能判别
console.log(typeof []); // object
console.log(typeof {}); // object
console.log(typeof null); // object
```

##### instanceof

能判断**对象**类型，不能判断基本数据类型，**其内部运行机制是判断在其原型链中能否找到该类型的原型**

`instanceof` 运算符用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上

使用如下：

```js
object instanceof constructor1
```

`object`为实例对象，`constructor`为构造函数

```js
class People {}
class Student extends People {}

const vortesnail = new Student();

console.log(vortesnail instanceof People); // true
console.log(vortesnail instanceof Student); // true
```



##### Object.prototype.toString.call()

所有原始数据类型都是能判断的，还有 **Error 对象，Date 对象**等

```js
Object.prototype.toString.call(2); // "[object Number]"
Object.prototype.toString.call(""); // "[object String]"
Object.prototype.toString.call(true); // "[object Boolean]"
Object.prototype.toString.call(undefined); // "[object Undefined]"
Object.prototype.toString.call(null); // "[object Null]"
Object.prototype.toString.call(Math); // "[object Math]"
Object.prototype.toString.call({}); // "[object Object]"
Object.prototype.toString.call([]); // "[object Array]"
Object.prototype.toString.call(function () {}); // "[object Function]"
```



> 在面试中有一个经常被问的问题就是：如何判断变量是否为数组？

```js
Array.isArray(arr); // true
arr.__proto__ === Array.prototype; // true
arr instanceof Array; // true
Object.prototype.toString.call(arr); // "[object Array]"
```



#### 手写深拷贝

[浅拷贝与深拷贝](/Users/owsl/Desktop/works/notes/JavaScript/浅拷贝与深拷贝.md)

> 并不是最终版本

```js
function deepClone(target, map = new Map()) {
  // 基本数据类型直接返回
  if (typeof target !== "object") {
    return target;
  }
  //对引用类型特殊处理
  //判断是数组还是对象
  const cloneTarget = Array.isArray(target) ? [] : {};
  //   防止循环引用导致栈溢出 已存在则直接返回
  if (map.get(target)) {
    return map.get(target);
  }
  //若不存在则存储进map
  map.set(target, cloneTarget);
  //递归深拷贝
  for (const key in target) {
    cloneTarget[key] = deepClone(target[key], map);
  }
  return cloneTarget;
}

const a = {
    name: '折叠鸭鸭',
    age: 22,
    hobbies: { sports: '篮球', tv: '雍正王朝' },
    works: ['2020', '2021']
}
a.key = a // 环引用
const b = deepClone(a)

console.log(b)
// {
//     name: '折叠鸭鸭',
//     age: 22,
//     hobbies: { sports: '篮球', tv: '雍正王朝' },
//     works: [ '2020', '2021' ],
//     key: [Circular]
// }
console.log(b === a) // false
```

#### new操作符都做了什么

- `new` 通过构造函数创建出来的实例可以访问到构造函数原型链中的属性（即实例与构造函数通过原型链连接了起来） (对应**步骤2**)

- `new` 通过构造函数创建出来的实例可以访问到构造函数中的属性和方法 (对应**步骤3**)

现在在构建函数中显式加上返回值，并且这个返回值是一个原始类型

```js
function Test(name) {
  this.name = name
  return 1
}
const t = new Test('xxx')
console.log(t.name) // 'xxx'
```

- 可以发现，构造函数中返回一个原始值，然而这个返回值并没有作用

下面在构造函数中返回一个对象

```js
function Test(name) {
  this.name = name
  console.log(this) // Test { name: 'xxx' }
  return { age: 26 }
}
const t = new Test('xxx')
console.log(t) // { age: 26 }
console.log(t.name) // 'undefined'
```

- 可以发现，构造函数如果返回值为一个对象，那么这个返回值会被正常使用 (对应**步骤4**)

> **手写new**

```js
function _new(func, ...args) {
  // 1.创建一个新对象
  let obj = {};
  // 2.将新对象的隐式原型指向构造函数的显式原型
  obj.__proto__ = func.prototype;
  // 3.执行构造函数，将属性和方法添加到新创建的新对象上
  let result = func.apply(obj, args)
  // 4.如果构造函数执行的结构返回的是一个对象，那么返回这个对象
  if (result && typeof (result) == 'object' || typeof (result) == 'fuction') {
    return result;
  }
  // 如果构造函数返回的不是一个对象，直接返回创建的新对象
  return obj;
}
```



#### a==1 && a==2 && a==3

[a==1 && a==2 && a==3](https://segmentfault.com/a/1190000012921114)

```js
const a = {
  num: 0,
  valueOf: function() {
    return this.num += 1
  }
};
//当进行==比较时，js会隐式转换类型，若要转换的是一个Object，JavaScript会调用valueOf()方法，所以修改valueOf方法可达成该目的
const equality = (a==1 && a==2 && a==3);
console.log(equality); // true
```

知识点：

- 隐式转换
- object的valueOf函数 

补充：

- 使用相等操作符，js会做类型转化
- 我们的对象每次调用valueOf()它的值会增加1

所以比较的时候我们每次都能得到true。



#### var let const

- 变量提升
- 暂时性死区
- 块级作用域
- 重复声明
- 修改声明的变量



#### this

> 当前执行上下文（global、function 或 eval）的一个属性

1.函数是否在new中调用(new绑定)，如果是，那么this绑定的是新创建的对象。

2.函数是否通过call,apply调用，或者使用了bind(即硬绑定)，如果是，那么this绑定的就是指定的对象。

3.函数是否在某个上下文对象中调用(隐式绑定)，如果是的话，this绑定的是那个上下文对象。一般是obj.foo()

4.如果以上都不是，那么使用默认绑定。如果在严格模式下，则绑定到undefined，否则绑定到全局对象。

5.如果把null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用时会被忽略，实际应用的是默认绑定规则。

6.如果是箭头函数，箭头函数的this继承的是外层代码块的this。



#### 执行上下文 作用域与作用域链

当 JavaScript 执行一段可执行代码时，会创建对应的执行上下文。对于每个执行上下文，都有三个重要属性：

- 变量对象（Variable object，VO）；
- 作用域链（Scope chain）；
- this

**执行上下文在运行时确定，随时可能改变；作用域在定义时就确定(词法作用域的特点)，并且不会改变**

作用域：规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。换句话说，作用域决定了代码区块中变量和其他资源的可见性（全局作用域、函数作用域、块级作用域）

作用域链：从当前作用域开始一层层往上找某个变量，如果找到全局作用域还没找到，就放弃寻找。这种层级关系就是作用域链。



#### 词法环境

>**Lexical Environment** : it's the internal js engine construct that holds **identifier-variable mapping**. (here **identifier** refers to the name of variables/functions, and **variable** is the reference to actual object [including function type object] or primitive value). A lexical environment also holds a reference to a **parent lexical environment**.

在 ES5 后，Scope 被替代为 Environment，Environment 取代了作用域，称为 **「Lexical Environment（词法环境）」**。

在 JavaScript 中，每个运行的函数，代码块 `{...}` 以及整个脚本，都有一个被称为 **词法环境（Lexical Environment）** 的内部（隐藏）的关联对象。

词法环境对象由两部分组成：

1. **环境记录（Environment Record）** —— 一个存储所有局部变量作为其属性（包括一些其他信息，例如 `this` 的值）的对象。
2. 对 **外部词法环境** 的引用，与外部代码相关联。

一个“变量”只是 **环境记录** 这个特殊的内部对象的一个属性，与当前正在执行的（代码）块/函数/脚本有关，“获取或修改变量”意味着“获取或修改词法环境的一个属性”。

#### 闭包

根据 MDN 中文的定义，闭包的定义如下：

> 函数与对其状态即**词法环境**（**lexical environment**）的引用共同构成**闭包**（**closure**）。也就是说，闭包可以让你从内部函数访问外部函数作用域。在JavaScript，函数在每次创建时生成闭包。

也可以这样说：

> 闭包是指那些能够访问自由变量的函数。 自由变量是指在函数中使用的，但既不是**函数参数**也不是**函数的局部变量**的**变量**。 闭包 = 函数 + 函数能够访问的自由变量。


**有许多精辟的总结**，帮助理解：

> - 闭包本质就是上级作用域内变量的生命周期，因为被下级作用域内引用，而没有被释放。就导致上级作用域内的变量，等到下级作用域执行完以后才正常得到释放
>
> - 函数声明的作用域与执行的作用域是不同的就叫闭包
>
> - 函数在定义时的词法作用域以外的地方被调用就会形成闭包。闭包使得函数可以继续访问定义时的词法作用域
>
> - 1.「函数」和「函数内部能访问到的变量」的总和，就是一个闭包
>   
>   2.函数和对其词法环境的引用捆绑在一起，这样的组合就是闭包

由于GC不能即使回收闭包里的变量，闭包会有弊端--滥用会造成内存泄漏

**从闭包、词法环境和垃圾回收机制来回答面试官**

>闭包是嵌套的内部函数以及对其周围状态(词法环境)的引用组成的一个整体，因为闭包引用了外部变量，会导致该外部变量不能及时的被标记清除的垃圾回收机制回收，因此，被引用的外部变量的生命周期得以延长；我们可以用这个特性来完成一些功能，比如将许多数据和功能封装到内部函数中，只向外暴露一个函数或者对象，比如get()。**闭包的应用是非常广泛的，比方我们常见的节流，防抖，函数科里化**
>
>一般函数的词法环境在函数返回后就被销毁，但是闭包会保存对创建时所在词法环境的引用，即便创建时所在的执行上下文被销毁，但创建时所在词法环境依然存在，以达到延长变量的生命周期的目的。
>
>使用闭包时，因为被引用的变量不能即使被回收，过多的不能回收的变量会导致内存泄漏；所以使用闭包时的注意事项便是及时释放，设为null，将其变为垃圾对象，手动回收掉变量。

#### 手写call apply bind

- call和apply

  ```js
  // 手写call   思路就是通过对象调用函数隐式绑定来将this的指向变更为obj
  Function.prototype._call = function (obj, ...arguments) {
    // 因为调用call时传入指向undefined或者null，this会指向window，所以这儿需处理一下
    obj = obj || window;
    // Symbol是唯一的，防止重名key
    const func = Symbol();
    // 将func设置为obj的属性，值为this，这样obj.func时this便指向obj
    obj[func] = this;
    // 执行，返回执行值
    return obj[func](...arguments);
  }
  
  // apply同理，就是参数是数组
  Function.prototype._apply = function (obj, args) {
    obj = obj || window;
    const func = Symbol();
    obj[func] = this;
    return obj[func](...args);
  }
  ```

- bind

  >`bind` 方法与 `call / apply` 最大的不同就是前者返回一个绑定上下文的**函数**，而后两者是**直接执行**了函数。
  >
  >一个绑定函数也能使用 new 操作符创建对象：这种行为就像把原函数当成构造器，提供的this值被忽略，同时调用时的参数被提供给模拟函数,所以需要处理这种情况

  ```js
  Function.prototype._bind = function (context) {
    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }
    // 保存bind函数的this，指向调用者
    var _self = this
    var args = Array.prototype.slice.call(arguments, 1);
  
    var fBound = function () {
      var bindArgs = Array.prototype.slice.call(arguments);
      // 当作为构造函数调用时，this 指向实例，此时结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值
      // 当作为普通函数调用时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
      return _self.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
    }
    // 若调用bind的函数不能作为构造函数时，会报错   this.prototype=undefined     所以这儿需做一下判断
    console.log(this.prototype);
    if (this.prototype) {
      fBound.prototype = Object.create(this.prototype)
    }
    return fBound; //返回一个函数
  }
  ```

[js 什么情况下函数的prototype是undefined](https://segmentfault.com/q/1010000016601164)



#### 0.1+0.2!==0.3

[硬核基础二进制篇（一）0.1 + 0.2 != 0.3 和 IEEE-754 标准](https://juejin.cn/post/6940405970954616839)

[0.1 + 0.2 不等于 0.3？为什么 JavaScript 有这种“骚”操作？](https://juejin.cn/post/6844903680362151950)。

JavaScript遵循IEEA754 使用64位双精度浮点数编码,包含1位符号位，11位阶数以及52位尾数

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20220503210410432.png" alt="image-20220503210410432" style="zoom:50%;" align="left" />

数字`0.1 0.2`在计算机里以二进制表示时，本身就会存在**精度丢失**(原本无限循环的数字被截断了)

之后进行**对阶运算**时，由于指数位数不相同，运算时需要对阶运算，阶小的尾数要根据阶差来右移（`0舍1入`），尾数位移时可能会发生数丢失的情况，影响精度



#### Event Loop事件循环

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20220913214808018.png" alt="image-20220913214808018" style="zoom:50%;" />



首先，整体的script(作为第一个宏任务)开始执行的时候，会把所有代码分为`同步任务`、`异步任务`两部分

同步任务会直接进入主线程依次执行

异步任务会再分为宏任务和微任务

宏任务进入到`Event Table`中，并在里面注册回调函数，每当指定的事件完成时，`Event Table`会将这个函数移到`Event Queue`中

微任务也会进入到另一个`Event Table`中，并在里面注册回调函数，每当指定的事件完成时，`Event Table`会将这个函数移到`Event Queue`中

当主线程内的任务执行完毕，主线程为空时，会检查微任务的`Event Queue`，如果有任务，就全部执行，如果没有就执行下一个宏任务

上述过程会不断重复，这就是`Event Loop`，比较完整的事件循环



#### 垃圾回收机制

- **标记清除**：标记阶段即为所有活动对象做上标记，清除阶段则把没有标记（也就是非活动对象）销毁。
- **引用计数**：它把**对象是否不再需要**简化定义为**对象有没有其他对象引用到它**。如果没有引用指向该对象（引用计数为 0），对象将被垃圾回收机制回收。

**标记清除**的缺点在于**内存碎片化**：空闲内存块是不连续的，容易出现很多空闲内存块，还可能会出现分配所需内存过大的对象时找不到合适的块以及**分配速度慢**

解决以上缺点可用**标记整理（Mark-Compact）算法**，标记结束后，标记整理算法会将活着的对象（即不需要清理的对象）向内存的一端移动，最后清理掉边界的内存。

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20220518221027611.png" alt="image-20220518221027611" style="zoom:50%;" />

**引用计数**的缺点：

- 需要一个计数器，所占内存空间大，因为我们也不知道被引用数量的上限。
- 解决不了循环引用导致的无法回收问题。

**v8引擎垃圾回收机制**也是基于标记清除算法，采用**分代式垃圾回收**

- 针对新生区采用并行回收
- 针对老生区采用增量标记与惰性回收
