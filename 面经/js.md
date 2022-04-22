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

<img src="/Users/owsl/Desktop/works/notes/图/继承.png" alt="继承" style="zoom:50%;" align="left"/>



#### 事件的传播

 事件的传播

关于事件的传播网景公司和微软公司有不同的理解
- 微软公司认为事件应该是由内向外传播，也就是当事件触发时，应该先触发当前元素上的事件，然后再向当前元素的祖先元素上传播，也就说事件应该在冒泡阶段执行。
  
- 网景公司认为事件应该是由外向内传播的，也就是当前事件触发时，应该先触发当前元素的最外层的祖先元素的事件，然后在向内传播给后代元素
  
- W3C综合了两个公司的方案，将事件传播分成了三个阶段
  
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
    if (typeof target !== 'object') {
        return target
    }
    // 引用数据类型特殊处理
    // 判断数组还是对象
    const cloneTarget = Array.isArray(target) ? [] : {}
    if (map.get(target)) {
         // 已存在则直接返回
         return map.get(target)
     }
     // 不存在则第一次设置 将当前对象作为key，克隆对象作为value进行存储
    map.set(target, cloneTarget)

    for (const key in target) {
        // 递归
        temp[key] = deepClone(target[key], map)
    }
    return temp
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

#### 手写new

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

作用域：规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。换句话说，作用域决定了代码区块中变量和其他资源的可见性（全局作用域、函数作用域、块级作用

域）

作用域链：从当前作用域开始一层层往上找某个变量，如果找到全局作用域还没找到，就放弃寻找 。这种层级关系就是作用域链。



#### 闭包

根据 MDN 中文的定义，闭包的定义如下：

> 在 JavaScript 中，每当创建一个函数，闭包就会在函数创建的同时被创建出来。可以在一个内层函数中访问到其外层函数的作用域。

也可以这样说：

> 闭包是指那些能够访问自由变量的函数。 自由变量是指在函数中使用的，但既不是**函数参数**也不是**函数的局部变量**的**变量**。 闭包 = 函数 + 函数能够访问的自由变量。


**有许多精辟的总结**，帮助理解：

> - 闭包本质就是上级作用域内变量的生命周期，因为被下级作用域内引用，而没有被释放。就导致上级作用域内的变量，等到下级作用域执行完以后才正常得到释放
>
> - 函数声明的作用域与执行的作用域是不同的就叫闭包
>
> - 函数在定义时的词法作用域以外的地方被调用就会形成闭包。闭包使得函数可以继续访问定义时的词法作用域
> - 1.「函数」和「函数内部能访问到的变量」的总和，就是一个闭包
>   2. 函数和对其词法环境的引用捆绑在一起，这样的组合就是闭包

闭包的弊端就是滥用会造成内存泄漏



