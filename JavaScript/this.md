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

> 注：只有foo()运行在非严格模式下时，默认绑定才能绑到全局对象,这时，严格模式下调用foo()不影响默认绑定             ----你不知道的js(上p84)

```js
function foo() {
  console.log(this); // window
}
var a = 2;

(fuction(){
 "use strict";
 foo(); //2
 })()
```

上面的代码foo运行在非严格模式下，但调用在严格模式下，不影响默认绑定

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

> 注：严格来说，foo函数不属于obj对象，可以说函数被调用时obj对象<mark>"拥有"或者"包含"</mark>这个**函数引用**；当函数引用有上下文对象时，**隐式绑定**规则会把函数调用中的this绑定到这个上下文对象。            ----你不知道的js(上p85)

##### 案例二：案例一的变化

- 我们通过obj2又引用了obj1对象，再通过obj1对象调用foo函数；
- 那么foo调用的位置上其实还是obj1被绑定了this；
- **对象属性链中只有最后一层会影响到调用位置**。

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

// 将obj1的foo赋值给bar
var bar = obj1.foo;
bar();
```

> 注：实际上var引用的是foo函数本身，因此此时的bar()其实是一个不带任何修饰的函数调用

**2.将函数作为参数，传入到另一个函数中**(回调函数)

- 参数传递其实就是一种隐式赋值

- 隐式绑定的丢失也会发生在回调函数中，foo(bar)执行时相当于把bar赋值给参数func，最后会执行这个func，这样只是一个独立函数的调用；

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
```

##### 硬绑定(bind)

硬绑定就是显式绑定的一个变种，可以解决之前提出的丢失绑定问题。

典型案例：

```js
function foo(sth) {
  return this.a + sth;
}
let obj = {
  a: 2
};
let bar = function () {
  return foo.apply(obj, arguments)
}
var b = bar(3);
console.log(b); // 5

/** 演变为↓ */
function foo(sth) {
  return this.a + sth;
}
let obj = {
  a: 2
};
function bind(fn, obj) {
  return function () {
    return fn.apply(obj, arguments)
  }
}
let bar = bind(foo, obj)
var b = bar(3);
console.log(b); // 5
```

于是ES5提供了内置方法`Function.prototype.bind`

```js
function foo() {
  console.log(this);
}

var obj = {
  name: "why"
}

var bar = foo.bind(obj);

bar(); // obj对象
```

#### 2.4 new绑定

> 注：JavaScript里的”构造函数“只是一些使用new操作符时被调用的函数，和其他一些面向类的语言完全不一样

使用new来调用函数的时候，就会新对象绑定到这个函数的this上。

会执行如下的操作：

- 1.创建(构造)一个全新的对象；
- 2.将空对象的隐式原型指向构造函数的显式原型(这个新对象会被执行Prototype连接)
- 3.这个新对象会绑定到函数调用的this上（this的绑定在这个步骤完成）；
- 4.如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象；

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

#### 衍生思考(手写bind--new中使用硬绑定--科里化)

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
    /** 这段代码会判断硬绑定函数是否被new调用，如果是的话会使用新创建的this替换硬绑定的this */
    // 当作为构造函数调用时，this 指向实例，此时结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值
    // 当作为普通函数调用时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
    return _self.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
  }
  // 若调用bind的函数不能作为构造函数(比如下面的obj.testFn)会报错     所以这儿需做一下判断
  if (this.prototype) {
    fBound.prototype = Object.create(this.prototype)
  }
  return fBound; //返回一个函数
}
```

之所以要在new中使用硬绑定函数，主要目的是<mark>预先设置函数的一些参数</mark>,这样在使用new进行初始化的时候就可以只传入其余的参数，是”**科里化**“的一种

```js
// a way of accomplishing Currying
function foo(p1, p2) {
  this.val = p1 + p2
}
// 使用null是因为本例中并不关心硬绑定的this是什么，反正new时会被修改
var bar = foo.bind(null, 2)
var test = new bar(3)
console.log(test.val); // 5
```



### 三.this的指向

**this 永远指向最后调用它的那个对象**

1.函数是否在new中调用(new绑定)，如果是，那么this绑定的是新创建的对象。

2.函数是否通过call,apply调用，或者使用了bind(即硬绑定)，如果是，那么this绑定的就是指定的对象。

3.函数是否在某个上下文对象中调用(隐式绑定)，如果是的话，this绑定的是那个上下文对象。一般是obj.foo()

4.如果以上都不是，那么使用默认绑定。如果在严格模式下，则绑定到undefined，否则绑定到全局对象。

5.如果把null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用时会被忽略，实际应用的是默认绑定规则。

6.如果是箭头函数，箭头函数的this继承的是外层代码块的this。

### 四.绑定例外

#### 被忽略的this

若把null或undefined作为this的绑定对象传入call.apply.bind，这些值在调用时会被忽略，实际应用的是默认绑定规则：

```javascript
function foo(){
  console.log(this.a)
}
var a = 2
foo.call(null) //2
```

常见的做法，比如展开数组和使用bind科里化

```js
function spread(a,b){
  console.log(a,b);  
}
spread.apply(null,[2,3]) //2 3

let curry = spread.bind(null,2)
curry(3) //2 3

spread.bind(null,2)(3) //2 3
```

##### [How does the Math.max.apply() work](https://stackoverflow.com/questions/21255138/how-does-the-math-max-apply-work)

`apply` accepts an array and it applies the array as parameters to the actual function. So,

```js
Math.max.apply(Math, list);
```

can be understood as,

```js
Math.max("12", "23", "100", "34", "56", "9", "233");
```

So, `apply` is a convenient way to pass an array of data as parameters to a function. Remember

```js
console.log(Math.max(list));   # NaN
```

will not work, because `max` doesn't accept an array as input.

There is another advantage, of using `apply`, you can choose your own context. The first parameter, you pass to `apply` of any function, will be the `this` inside that function. But, `max` doesn't depend on the current context. So, anything would work in-place of `Math`.

```js
console.log(Math.max.apply(undefined, list));   # 233
console.log(Math.max.apply(null, list));        # 233
console.log(Math.max.apply(Math, list));        # 233
```

### 五.改变 this 的指向

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

因为：“<mark>箭头函数中没有 this 绑定，必须通过查找作用域链来决定其值，如果箭头函数被非箭头函数包含，则 this 绑定的是最近一层非箭头函数的 this，否则，this为undefined</mark>”。

函数体内的this对象，继承的是外层代码块的this。

机制类似于下面的第二条

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





