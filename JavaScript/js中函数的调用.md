[toc]

函数调用的方法一共有 4 种

1. 作为一个函数调用
2. 函数作为方法调用
3. 使用构造函数调用函数
4. 作为函数方法调用函数（call、apply）

### 作为一个函数调用

比如：

```js
var name = "windowsName";
function a() {
  var name = "Cherry";

  console.log(this.name);          // windowsName

  console.log("inner:" + this);    // inner: Window
}
a();
console.log("outer:" + this)         // outer: Window
```

这样一个最简单的函数，不属于任何一个对象，就是一个函数，这样的情况在 JavaScript 的在浏览器中的非严格模式默认是属于全局对象 window 的

但是在严格模式，就是 undefined。 

但这是一个全局的函数，很容易产生命名冲突，所以不建议这样使用。

### 函数作为方法调用

比如：

```js
var name = "windowsName";
var a = {
  name: "Cherry",
  fn : function () {
     console.log(this.name);      // Cherry
  }
}
a.fn();
```

这里定义一个对象`a`，对象`a`有一个属性（`name`）和一个方法（`fn`）。

然后对象`a`通过`.`方法调用了其中的 fn 方法。

然后我们一直记住的那句话“**this 永远指向最后调用它的那个对象**”，所以在 fn 中的 this 就是指向 a 的。

### 使用构造函数调用函数

>如果函数调用前使用了 new 关键字, 则是调用了构造函数。
>这看起来就像创建了新的函数，但实际上 JavaScript 函数是重新创建的对象：

```js
// 构造函数:
function myFunction(arg1, arg2) {
    this.firstName = arg1;
    this.lastName  = arg2;
}

// This    creates a new object
var a = new myFunction("Li","Cherry");
a.lastName;                             // 返回 "Cherry"
```

简单的来看一下 new 的过程：

伪代码表示：

```js
var a = new myFunction("Li","Cherry");

new myFunction{
    var obj = {};
    obj.__proto__ = myFunction.prototype;
    var result = myFunction.call(obj,"Li","Cherry");
    return typeof result === 'obj'? result : obj;
}
```

1. 创建一个空对象 obj;
2. 将新创建的空对象的隐式原型指向其构造函数的显式原型。
3. 使用 call 改变 this 的指向
4. 如果无返回值或者返回一个非对象值，则将 obj 返回作为新对象；如果返回值是一个新对象的话那么直接直接返回该对象。

所以我们可以看到，在 new 的过程中，我们是使用 call 改变了 this 的指向。

### 作为函数方法调用函数

> 在 JavaScript 中, 函数是对象。
>
> JavaScript 函数有它的属性和方法。
> call() 和 apply() 是预定义的函数方法。 两个方法可用于调用函数，两个方法的第一个参数必须是对象本身
>
> 在 JavaScript 严格模式(strict mode)下, 在调用函数时第一个参数会成为 this 的值， 即使该参数不是一个对象。
> 在 JavaScript 非严格模式(non-strict mode)下, 如果第一个参数的值是 null 或 undefined, 它将使用全局对象替代。

如：

```js
var name = "windowsName";

function fn() {
  var name = 'Cherry';
  innerFunction();
  function innerFunction() {
    console.log(this.name);      // windowsName
  }
}

fn()复制代码
```

这里的 innerFunction() 是作为一个函数调用的，没有挂载在任何对象上，所以对于没有挂载在任何对象上的函数，在非严格模式下 this 就是指向 window 的



setTimeout

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

当传递函数时this如果不经过显式和硬性绑定 是会在global下execute的

函数作为回调或者参数时，根据隐式绑定规则，this会绑定丢失，所以是默认绑定，就是window

setTimeout回调函数会经历 Event Loop => Execution Stack => Execute，this指向window，是因为 Execute 在 Global Execution Context下执行



#### 回调函数

被作为实参传入另一函数，并在该外部函数内被调用，用以来完成某些任务的函数，称为回调函数。

> *回调函数*作为*高阶函数*的参数，高阶函数通过调用回调函数来执行操作。
