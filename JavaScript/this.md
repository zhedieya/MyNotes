[toc]

### 一.前提

所有的函数在被调用时，都会创建一个执行上下文：

- 这个上下文中记录着函数的调用栈、函数的调用方式、传入的参数信息等；
- this也是其中的一个属性；



### 二.this绑定规则

#### 2.1 默认绑定

什么情况下使用默认绑定呢？独立函数调用。

- 独立的函数调用我们可以理解成函数没有被绑定到某个对象上进行调用；

##### 案例一：普通函数调用

```js
function foo() {
  console.log(this); // window
}
foo();
```

##### 案例二：函数调用链（一个函数又调用另外一个函数）

```js
function test1() {
  console.log(this); // window
  test2();
}
function test2() {
  console.log(this); // window
  test3()
}
function test3() {
  console.log(this); // window
}
test1();
```

#### 2.2 隐式绑定

另外一种比较常见的调用方式是通过某个对象进行调用的：

- 也就是它的调用位置中，是通过某个对象发起的函数调用。

##### 案例一：通过对象调用函数

- foo的调用位置是obj.foo()方式进行调用的
- 那么foo调用时this会隐式的被绑定到obj对象上

```js
function foo() {
  console.log(this); // obj对象
}
var obj = {
  name: "why",
  foo: foo
}
obj.foo();
```

##### 案例二：案例一的变化

- 我们通过obj2又引用了obj1对象，再通过obj1对象调用foo函数；
- 那么foo调用的位置上其实还是obj1被绑定了this；
- 对象属性链中只有最后一层会影响到调用位置。

```js
function foo() {
  console.log(this); // obj对象
}
var obj1 = {
  name: "obj1",
  foo: foo
}
var obj2 = {
  name: "obj2",
  obj1: obj1
}
obj2.obj1.foo();
```

##### 案例三：隐式丢失

**1.函数别名**

- 结果最终是window
- 因为foo最终被调用的位置是bar，而bar在进行调用时没有绑定任何的对象，也就没有形成隐式绑定；
- 相当于是一种默认绑定；

```js
function foo() {
  console.log(this);
}

var obj1 = {
  name: "obj1",
  foo: foo
}

// 讲obj1的foo赋值给bar
var bar = obj1.foo;
bar();
```

**2.将函数作为参数，传入到另一个函数中** 

隐式绑定的丢失也会发生在回调函数中，foo(bar)执行时相当于把bar赋值给参数func，最后会执行这个func，这样只是一个独立函数的调用；

```js
function foo(func) {
  func()
}

function bar() {
  console.log(this); // window
}

foo(bar);
//相当于 foo(window.bar)
```

再看一个例子

```js
function foo(func) {
  func()
}

function bar() {
  console.log(this); // window
}

var obj={
  name:'obj',
  bar:bar
}

foo(obj.bar);
```

上面的代码 把`obj.bar`当作参数传递给`foo`函数时，在`foo`函数里，有隐式的函数赋值`func=obj.bar`。只是把`bar`函数赋给了`func`，而`func`与`obj`对象则毫无

关系。

 **3.setTimeout 等内置函数**

```js
setTimeout(function() {
  console.log(this); // window
}, 1000);
```

#### 2.3 显式绑定

##### 通过call或者apply绑定this对象

- 显式绑定后，this就会明确的指向绑定的对象

```js
function foo() {
  console.log(this);
}

foo.call(window); // window
foo.call({name: "why"}); // {name: "why"}
foo.call(123); // Number对象,存放123
```

##### 通过bind来绑定this读写

如果我们希望一个函数总是显式的绑定到一个对象上，可以怎么做呢？

方案一：自己手写一个辅助函数（了解）

- 手动写一个bind的辅助函数
- 这个辅助函数的目的是在执行foo时，总是让它的this绑定到obj对象上

```js
function foo() {
  console.log(this);
}
var obj = {
  name: "why"
}
function bind(func, obj) {
  return function() {
    return func.apply(obj, arguments);
  }
}
var bar = bind(foo, obj);

bar(); // obj对象
bar(); // obj对象
bar(); // obj对象
```

方案二：使用`Function.prototype.bind`

```js
function foo() {
  console.log(this);
}

var obj = {
  name: "why"
}

var bar = foo.bind(obj);

bar(); // obj对象
bar(); // obj对象
bar(); // obj对象
```



#### 2.4 new绑定

JavaScript中的函数可以当做一个类的构造函数来使用，也就是使用new关键字。

使用new来调用函数的时候，就会新对象绑定到这个函数的this上。

会执行如下的操作：

- 1.定义一个空对象；
- 2.将空对象的隐式原型指向构造函数的显式原型
- 3.这个新对象会绑定到函数调用的this上（this的绑定在这个步骤完成）；
- 4.返回新对象；

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

🌰：

```js
// 创建Person
function Person(name) {
  console.log(this); // Person {}
  this.name = name; // Person {name: "why"}
}

var p = new Person("why");
console.log(p);
```

### 三.this的指向

**this 永远指向最后调用它的那个对象**

1.函数是否在new中调用(new绑定)，如果是，那么this绑定的是新创建的对象。

2.函数是否通过call,apply调用，或者使用了bind(即硬绑定)，如果是，那么this绑定的就是指定的对象。

3.函数是否在某个上下文对象中调用(隐式绑定)，如果是的话，this绑定的是那个上下文对象。一般是obj.foo()

4.如果以上都不是，那么使用默认绑定。如果在严格模式下，则绑定到undefined，否则绑定到全局对象。

5.如果把null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用时会被忽略，实际应用的是默认绑定规则。

6.如果是箭头函数，箭头函数的this继承的是外层代码块的this。

### 四.改变 this 的指向

- 使用 ES6 的箭头函数
- 在函数内部使用 `_this = this`
- 使用 `apply`、`call`、`bind`
- new 实例化一个对象

以该函数为例：

```js
var name = "windowsName";
var a = {
  name : "Cherry",
  func1: function () {
    console.log(this.name)     
  },

  func2: function () {
    setTimeout(function () {
      this.func1()
    },100 );
  }
};
a.func2()     // this.func1 is not a function
```

接下来会利用各种方法更改该函数的this

#### 1.箭头函数

ES6的箭头函数的this会始终指向函数定义时的this，而非执行时。

因为：“箭头函数中没有 this 绑定，必须通过查找作用域链来决定其值，如果箭头函数被非箭头函数包含，则 this 绑定的是最近一层非箭头函数的 this，否则，this为

undefined”。

函数体内的this对象，继承的是外层代码块的this。

```js
func2: function () {
  setTimeout(() => {
    this.func1()
  },100);
}

a.func2() // Cherry
```

#### 2.在函数内部使用_this=this

将调用该函数的对象保存到`_this`上，在函数中都使用这个`_this`，这样`_this`就不会改变了。

```js
func2: function () {
  var _this = this;
  setTimeout(function() {
    _this.func1()
  },100);
}

a.func2() // Cherry
```

在func2中，设置`var _this = this;`，这里的`this`是调用`func2`的对象，将 `this`(指向变量 a) 赋值给一个变量`_this`，这样，在`func2`中

 `_this` 就永远指向对象a了。

#### 3.使用 apply、call、bind

```js
//apply
func2: function () {
  setTimeout(function () {
    this.func1()
  }.apply(a),100);
}
//call
func2: function () {
  setTimeout(function () {
    this.func1()
  }.call(a),100);
}
//bind
func2: function () {
  setTimeout(function () {
    this.func1()
  }.bind(a)(),100);
}
```

浏览器中运行是可以的；在node里报错了TypeError: "callback" argument must be a function

因为：

1、浏览器的setTimeout还有语法，可以传字符串，它会像 eval一样执行

2、node重写了setTimeout，对参数进行检查不能传字符串

3、因为function () {this.func1()}.call(a)会直接执行，传给setTimeout是个字符串而不是一个function，所以node环境会直接报错

##### 区别

**`apply()`** 方法调用一个具有给定`this`值的函数，以及作为一个数组（或[类似数组对象](6eb909c3e8d2512c06e71e38bc16d5b1.html#Working_with_array-like_objects)）提供的参数。

```
func.apply(thisArg, [argsArray])
```



**`call()`**方法作用和apply()方法类似，区别就是`call()`方法接受的是**参数列表**，而`apply()`方法接受的是**一个参数数组**。

```js
fun.call(thisArg[, arg1[, arg2[, ...]]])
```

🌰：

```js
var a ={
  name : "Cherry",
  fn : function (a,b) {
    console.log( a + b)
  }
}
var b = a.fn;

b.apply(a,[1,2]) //apply 3
b.call(a,1,2) // call 3
```



**`bind()`**方法创建一个**新的函数**，在`bind()`被调用时，这个新函数的`this`被bind的第一个参数指定，其余的参数将作为新函数的参数供调用时使用。

```
function.bind(thisArg[,arg1[,arg2[, ...]]])
```

🌰：

```js
var a ={
  name : "Cherry",
  fn : function (a,b) {
    console.log( a + b)
  }
}

var b = a.fn;
b.bind(a,1,2)()  // 3
```





