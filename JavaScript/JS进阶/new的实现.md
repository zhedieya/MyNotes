[toc]

### new 运算符实现

简而言之：new 运算符 创建一个用户定义的对象类型的实例 或 具有构造函数的内置对象类型之一

举个例子：

```js
// Otaku 御宅族，简称宅
function Otaku (name, age) {
    this.name = name;
    this.age = age;
    this.habit = 'Games';
}

// 因为缺乏锻炼的缘故，身体强度让人担忧
Otaku.prototype.strength = 60;

Otaku.prototype.sayYourName = function () {
    console.log('I am ' + this.name);
}

var person = new Otaku('Kevin', '18');

console.log(person.name) // Kevin
console.log(person.habit) // Games
console.log(person.strength) // 60

person.sayYourName(); // I am Kevin
```

根据示例可知，new 创建的 person 实例可以：

- 访问 Otaku 构造函数里的属性
- 访问到 Otaku.prototype 中的属性

##### 接下来，我们可以尝试着模拟一下了

<mark>因为 new 是关键字，所以无法像 bind 函数一样直接覆盖</mark>，所以我们写一个函数，命名为 objectFactory，来模拟 new 的效果。用的时候是这样的：

```js
function Otaku () { …… }

// 使用 new
var person = new Otaku(……);
// 使用 objectFactory
var person = objectFactory(Otaku, ……) // 注：第一个参数为“构造函数”名
```

#### 初步实现

##### 分析

因为 new 的结果是一个新对象，所以在模拟实现的时候，我们也要建立一个新对象，假设这个对象叫 obj，因为 obj 会具有 Otaku 构造函数里的属性，<font color=FF0000>想想**「经典继承」**的例子</font>，我们<font color=FF0000>**可以使用 Otaku.apply(obj, arguments)来给 obj 添加新的属性**</font>。

在 [JavaScript 深入系列第一篇：JavaScript深入之从原型到原型链](https://github.com/mqyqingfeng/Blog/issues/2) 中，我们便讲了原型与原型链，我们知道实例的 \_\_proto__ 属性会指向构造函数的 prototype，也正是因为建立起这样的关系，实例可以访问原型上的属性。

现在，我们可以尝试着写第一版了：

```js
// 第一版代码
function objectFactory() {
    // 注：创建一个新的对象，最后返回
    var obj = new Object();
    // 注：弹出并使用 第一个实参（构造函数名），用名为 Constructor 的变量保存起来。另外，这里的 [].shift.call() 一般写作 Array.prototype.shift.call()；[].shift.call() 有点不易读
    Constructor = [].shift.call(arguments);
    // 注：将 Contructor 构造函数的原型 赋值到 创建的对象 obj 的原型上。这里 obj.__proto__ 是 ES5 的写法，已经不推荐使用；可以使用 ES6 的 Object.setPrototypeOf() 替代。另外，new 本身也是 ES6 的语法，不会出现不兼容的情况。
    obj.__proto__ = Constructor.prototype;
    // 注：使用 apply 调用 contructor 函数
    Constructor.apply(obj, arguments);
    // 返回 obj
    return obj;
};
```

在这一版中，我们：

1. 用new Object() 的方式新建了一个对象 obj
2. 取出第一个参数，就是我们要传入的构造函数。此外因为 shift 会修改原数组，所以 arguments 会被去除第一个参数
3. 将 obj 的原型指向构造函数，这样 obj 就可以访问到构造函数原型中的属性
4. 使用 apply，改变构造函数 this 的指向到新建的对象，这样 obj 就可以访问到构造函数中的属性
5. 返回 obj

复制以下的代码，到浏览器中，我们可以做一下测试：

```js
function Otaku (name, age) {
    this.name = name;
    this.age = age;
    this.habit = 'Games';
}

Otaku.prototype.strength = 60;

Otaku.prototype.sayYourName = function () {
    console.log('I am ' + this.name);
}

function objectFactory() {
    var obj = new Object(),
    Constructor = [].shift.call(arguments);
    obj.__proto__ = Constructor.prototype;
    Constructor.apply(obj, arguments);
    return obj;
};

var person = objectFactory(Otaku, 'Kevin', '18')

console.log(person.name) // Kevin
console.log(person.habit) // Games
console.log(person.strength) // 60

person.sayYourName(); // I am Kevin
```

#### 返回值效果实现

**注：**下面的内容讲的有点唐突，可以看下下面的补充 [[#视频《new实例化的重写--检测一下自己this 指向？？》的补充#不同方法返回值区别]] 中 不同返回值的总结

接下来我们再来看一种情况，假如构造函数有返回值，举个例子：

```js
function Otaku (name, age) {
    this.strength = 60;
    this.age = age;
  
    return {
        name: name,
        habit: 'Games'
    }
}

var person = new Otaku('Kevin', '18');

console.log(person.name) // Kevin
console.log(person.habit) // Games
console.log(person.strength) // undefined 注：不是返回的属性，拿不到
console.log(person.age) // undefined      注：不是返回的属性，拿不到
```

在这个例子中，构造函数返回了一个对象，在 <font color=FF0000>**实例 person 中只能访问返回的对象中的属性**</font>。

而且还要注意一点，在这里我们是返回了一个对象，<font color=FF0000>假如我们只是返回一个基本类型的值呢</font>？再举个例子：

```js
function Otaku (name, age) {
    this.strength = 60;
    this.age = age;
  
    return 'handsome boy';
}

var person = new Otaku('Kevin', '18');

console.log(person.name) // undefined
console.log(person.habit) // undefined
console.log(person.strength) // 60
console.log(person.age) // 18
```

结果完全颠倒过来，<font color=FF0000>这次尽管有返回值，但是相当于没有返回值进行处理</font>。

<font color=FF0000>**所以我们还需要判断返回的值是不是一个对象**</font>，<font color=FF0000>**如果是一个对象，我们就返回这个对象**</font>；<font color=FF0000>**如果没有，我们该返回什么就返回什么**</font>。

再来看第二版的代码，也是最后一版的代码：

```js
// 第二版的代码
function objectFactory() {
    var obj = new Object(),
    Constructor = [].shift.call(arguments);
    obj.__proto__ = Constructor.prototype;
    var ret = Constructor.apply(obj, arguments);
    // 注：只有这里改变了，加了判断
    return typeof ret === 'object' ? ret : obj;
};
```

摘自：[JavaScript深入之new的模拟实现](https://github.com/mqyqingfeng/Blog/issues/13)

#### 视频《new实例化的重写--检测一下自己this 指向？？》的补充

```js
function Person(name) {
  this.name = name
}
```

如上 Person 构造函数 有两种执行方法：1) 直接函数执行（直接调用 `Person()` ） 2) 使用 new 运算符执行 `new Person()`。

##### 不同方法返回值区别

- **直接函数执行：**返回的内容要看 函数内容 return 的内容：有返回值则有值，没有返回值则为 undefined

- **使用 new 运算符：**一般没有 return 返回值，且返回的内容是实例之后的对象。

  有 return：如果 return 的是一个对象（引用值），则返回该对象；如果 return 的是「原始类型」，则忽略，返回当前的实例对象。

##### new 的实现

```js
// new 的实现
Function.prototype.new = function() {
  var fn = this
  var obj = Object.create(fn.prototype)
  var res = fn.apply(obj, arguments)
  return res instanceof Object ? res : obj
}

// 构造函数
function Person(name) {
  this.name = name
}
Person.prototype.say = function() {
  console.log('hello')
}

// 检验效果
var p = Person.new('foo')
console.log(p)
p.say()
```

学习自：[【全网首发:更新完】new实例化的重写--检测一下自己this 指向？？](https://www.bilibili.com/video/BV16L411u7kh) 另外，这类源码重写的视频组成了一个合集：[测试一下前端基本功--简单的源码重写？](https://www.bilibili.com/video/BV1dS4y1X7pn)

