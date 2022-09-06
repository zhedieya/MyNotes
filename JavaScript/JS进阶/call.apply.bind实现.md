[toc]

### call 和 apply 实现

#### call 的模拟实现

一句话介绍 call：<mark>call() 方法在使用一个 
**指定的 this 值** 和 **若干个指定的参数值** 的前提下调用某个函数或方法</mark>。

举个例子：

```js
var foo = { value: 1 };
function bar() { console.log(this.value); }

bar.call(foo); // 1
```

注意两点：

1. call 改变了 this 的指向，指向到 foo
2. bar 函数执行了

##### 模拟第一步

那么我们该怎么模拟实现这两个效果呢？

试想当调用 call 的时候，把 foo 对象改造成如下：

```js
var foo = {
    value: 1,
    bar: function() {
        console.log(this.value)
    }
};

foo.bar(); // 1
```

这个时候 this 就指向了 foo，是不是很简单呢？

但是这样却<font color=FF0000>给 foo 对象本身添加了一个属性</font>，这可不行呐！不过也不用担心，（注：在执行完毕后）我们<font color=FF0000>用 delete 再删除它</font>不就好了~

**所以我们模拟的步骤可以分为：**

1. 将函数设为对象的属性
2. <font color=FF0000>执行该函数</font>
3. <font color=FF0000>**删除该函数**</font>

以上个例子为例，就是：

```js
// 第一步
foo.fn = bar
// 第二步
foo.fn()
// 第三步
delete foo.fn
```

fn 是对象的属性名，反正最后也要删除它，所以起成什么都无所谓。

根据这个思路，我们可以尝试着去写第一版的 **myCall** 函数：

```js
// 第一版
Function.prototype.myCall = function(context) {
    // 首先要获取调用 call 的函数，用 this 可以获取。注：根据下面的调用来看，这里是 this 指向的是 bar
    context.fn = this;
    context.fn();
    delete context.fn;
}

// 测试
var foo = { value: 1 };
function bar() { console.log(this.value); }

bar.myCall(foo); // 1
```

##### 模拟实现第二步

最一开始也讲了，<font color=FF0000>call 函数还能给定参数执行函数</font>。举个例子：

```js
var foo = { value: 1 };

function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}

bar.call(foo, 'kevin', 18); // kevin 18 1
```

**注意：**传入的参数并不确定，这可咋办？我们可以从 Arguments 对象中取值，取出第二个到最后一个参数，然后放到一个数组里。

比如这样：

```js
// 以上个例子为例，此时的arguments为：
// arguments = {
//      0: foo,
//      1: 'kevin',
//      2: 18,
//      length: 3
// }
// 因为arguments是类数组对象，所以可以用 for 循环
var args = [];
for(var i = 1, len = arguments.length; i < len; i++) {
    args.push('arguments[' + i + ']');
}
// 执行后 args为 ["arguments[1]", "arguments[2]", "arguments[3]"]。注：这一步是为下面的 eval 作准备。
```

不定长的参数问题解决了，我们接着要把这个参数数组放到要执行的函数的参数里面去。

```js
// 将数组里的元素作为多个参数放进函数的形参里
context.fn( args.join(',') ); // 这个方法肯定是不行的啦！！！
```

也许有人想到用 ES6 的方法（**注：**比如 “展开表达式”，结果测试确实可行 ），不过 call 是 ES3 的方法，我们为了模拟实现一个 ES3 的方法，要用到ES6的方法，好像……，嗯，也可以啦。但是我们这次用 eval 方法拼成一个函数，类似于这样：

```js
eval('context.fn(' + args +')')
```

<font color=FF0000>这里 args 会自动调用 Array.toString() 这个方法</font>。

所以我们的第二版克服了两个大问题，代码如下：

```js
// 第二版
Function.prototype.call2 = function(context) {
    context.fn = this;
    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }
    eval('context.fn(' + args +')');
    delete context.fn;
}

// 测试一下
var foo = { value: 1 };
function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}
bar.call2(foo, 'kevin', 18); // kevin, 18, 1
```

##### 模拟实现第三步

模拟代码已经完成 80%，还有两个小点要注意：

1. **this 参数可以传 null，当为 null 的时候，视为指向 window。**举个例子：

   ```js
   var value = 1;
   function bar() { console.log(this.value); }
   
   bar.call(null); // 1
   ```

   虽然这个例子本身不使用 call，结果依然一样。

2. **函数是可以有返回值的。**举个例子：

   ```js
   var obj = { value: 1 }
   
   function bar(name, age) {
       return {
           value: this.value,
           name: name,
           age: age
       }
   }
   
   console.log(bar.call(obj, 'kevin', 18)); // Object { value: 1,name: 'kevin',age: 18 }
   ```

   不过都很好解决，让我们直接看第三版也就是最后一版的代码：

   ```js
   // 第三版
   Function.prototype.call2 = function (context) {
       var context = context || window;
       context.fn = this;
   
       var args = [];
       for(var i = 1, len = arguments.length; i < len; i++) {
           args.push('arguments[' + i + ']');
       }
       var result = eval('context.fn(' + args +')');
   
       delete context.fn
       return result; // 返回了运行的返回值
   }
   
   // 测试一下
   var value = 2;
   var obj = { value: 1 }
   
   function bar(name, age) {
       console.log(this.value);
       return {
           value: this.value,
           name: name,
           age: age
       }
   }
   
   bar.call2(null); // 2
   console.log(bar.call2(obj, 'kevin', 18)); // 1, { value: 1, name: 'kevin', age: 18 }
   ```

#### apply 的模拟实现

apply 的实现跟 call 类似，在这里直接给代码，代码来自于知乎 @郑航 的实现：

```js
Function.prototype.apply = function (context, arr) {
    var context = Object(context) || window;
    context.fn = this;

    var result;
    if (!arr) {
        result = context.fn();
    }
    else {
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) { // 和 for (var i = 0; i < arr.length; i++) 有区别吗？
            args.push('arr[' + i + ']');
        }
        result = eval('context.fn(' + args + ')')
    }

    delete context.fn
    return result;
}
```

摘自：[JavaScript深入之call和apply的模拟实现 ](https://github.com/mqyqingfeng/Blog/issues/11)

### bind 模拟实现

**一句话介绍 bind：**bind() 方法会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数。（来自于 MDN )

**由此我们可以首先得出 bind 函数的两个特点：**

1. 返回一个函数
2. 可以传入参数

#### 返回函数的模拟实现

从第一个特点开始，我们举个例子：

```js
var foo = { value: 1 };
function bar() { console.log(this.value); }

// 返回了一个函数
var bindFoo = bar.bind(foo); 
bindFoo(); // 1
```

关于指定 this 的指向，我们可以使用 call 或者 apply 实现，关于 call 和 apply 的模拟实现，可以查看[《JavaScript深入之call和apply的模拟实现》](https://github.com/mqyqingfeng/Blog/issues/11)。我们来写第一版的代码：

```js
// 第一版
Function.prototype.bind = function (context) {
    var self = this; // 谁调用的，也就是前面的要被 bind 函数绑定 this值 的 函数。
    return function () {
        return self.apply(context);
    }

}
```

此外，之所以 `return self.apply(context)`，是<font color=FF0000>考虑到绑定函数可能是有返回值的</font>。依然是这个例子：

```js
var foo = { value: 1 };
function bar() { return this.value; }

var bindFoo = bar.bind(foo);
console.log(bindFoo()); // 1
```

#### 传参的模拟实现

接下来看第二点，可以传入参数。这个就有点让人费解了，我在 bind 的时候，是否可以传参呢？我在执行 bind 返回的函数的时候，可不可以传参呢？让我们看个例子：

```js
var foo = { value: 1 };
function bar(name, age) {
    console.log(this.value);
    console.log(name);
    console.log(age);
}

var bindFoo = bar.bind(foo, 'daisy');
bindFoo('18'); // 1, daisy, 18
```

函数需要传 name 和 age 两个参数，竟然还可以在 bind 的时候，只传一个 name，在执行返回的函数的时候，再传另一个参数 age。这可以用 arguments 进行处理：

```js
// 第二版
Function.prototype.bind = function (context) {
    var self = this;
    // 获取 bind 函数从第二个参数到最后一个参数
    var args = Array.prototype.slice.call(arguments, 1);

    return function () {
        // 这个时候的arguments是指bind返回的函数传入的参数
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(context, args.concat(bindArgs));
    }

}
```

#### 构造函数效果的模拟实现

完成了这两点，<font color=FF0000>最难的部分到啦</font>！因为 <font color=FF0000>bind 还有一个特点</font>，就是

> 一个绑定函数也能使用 new 操作符创建对象：这种行为就像把原函数当成构造器（即：Constructor）。<font color=FF0000>**提供的 this 值被忽略**</font>，同时调用时的参数被提供给模拟函数。

也就是说当 bind 返回的函数作为构造函数的时候，bind 时指定的 this 值会失效，但传入的参数依然生效（**注：**这里可以参考 [[前端面试点总结#this的绑定（学习自 coderwhy 的文章）]] 中的：<font color=FF0000>**bind 的优先级没有 new 高**</font>）。举个例子：

```js
var value = 2;
var foo = { value: 1 };

function bar(name, age) {
    this.habit = 'shopping';
    console.log(this.value); // 这里的 this 指向的是 bar {habit: 'shopping'}，而不是obj，所以obj.value === undefined
    console.log(name);
    console.log(age);
}
bar.prototype.friend = 'kevin';
var bindFoo = bar.bind(foo, 'daisy');

var obj = new bindFoo('18'); // undefined, daisy, 18
console.log(obj.habit); // shopping
console.log(obj.friend); // kevin
```

**注意：**<font color=FF0000>尽管在全局和 foo 中都声明了 value 值，最后依然返回了 undefind，**说明绑定的 this 失效了**</font>；如果大家了解 new 的模拟实现，就会知道这个时候的 this 已经指向了 obj。

所以我们可以通过修改返回的函数的原型来实现，让我们写一下：

```js
// 第三版
Function.prototype.bind = function (context) {
    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fBound = function () {
        // 注：这里的 arguments 和上面的 arguments 不同一次调用获取的，也就是说：不是一个东西。
        var bindArgs = Array.prototype.slice.call(arguments);
        // 当作为构造函数时，this 指向实例，此时结果为 true；将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值。**注：**个人理解，作为构造函数使用时：this 在 new 操作符的作用下，指向实例；另外，此时 fBound 就是指定 this 指向后 的“构造函数”。
        // 以上面的是 demo 为例，如果改成 `this instanceof fBound ? null : context`，实例只是一个空对象，将 null 改成 this ，实例会具有 habit 属性
        // 当作为普通函数时，this 指向 window，此时（**注：**this instanceof fBound 的 ）结果为 false，将绑定函数的 this 指向 context
        return self.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
    }
    
    // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值。
    fBound.prototype = this.prototype;
    return fBound; // 注：返回 fBound 函数
}
```

#### 构造函数效果的优化实现

但是在这个写法中，我们直接将 fBound.prototype = this.prototype，我们直接修改 fBound.prototype 的时候，也会直接修改绑定函数的 prototype。这个时候，我们可以通过一个空函数来进行中转：

```js
// 第四版
Function.prototype.bind = function (context) {

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {}; // 注：NOP 的意思是 No Operation，意为无操作。是汇编语言的一个指令，一系列编程语句，或网络传输协议中的表示不做任何有效操作的命令。

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
```

#### 三个小问题

接下来处理些小问题：

##### 一：这一点如今找不到了，略。

##### 二：调用 bind 的不是函数咋办？

不行，我们要报错！

```js
if (typeof this !== "function") {
  throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
}
```

##### 要在线上用

那别忘了做个兼容：

```js
Function.prototype.bind = Function.prototype.bind || function () { ... };
```

当然最好是用 [es5-shim](https://github.com/es-shims/es5-shim) 啦。

#### 最终代码

```js
Function.prototype.bind2 = function (context) {

    // 比上一个版本 多了一个校验判断
    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {};

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
```

摘自：[JavaScript深入之bind的模拟实现](https://github.com/mqyqingfeng/Blog/issues/12)