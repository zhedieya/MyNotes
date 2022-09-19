[toc]

### 类数组对象

所谓的类数组对象：拥有一个 length 属性和若干索引属性的对象。

举个例子：

```js
var array = ['name', 'age', 'sex'];

var arrayLike = {
    0: 'name',
    1: 'age',
    2: 'sex',
    length: 3
}
```

即便如此，为什么叫做类数组对象呢？那让我们从读写、获取长度、遍历三个方面看看这两个对象。

##### 读写

```js
console.log(array[0]); // name
console.log(arrayLike[0]); // name

array[0] = 'new name';
arrayLike[0] = 'new name';
```

##### 长度

```js
console.log(array.length); // 3
console.log(arrayLike.length); // 3
```

##### 遍历

```js
for(var i = 0, len = array.length; i < len; i++) { …… }
for(var i = 0, len = arrayLike.length; i < len; i++) { …… }
```

是不是很像？

那类数组对象可以使用数组的方法吗？比如：`arrayLike.push('4');` 然而上述代码会报错：arrayLike.push is not a function。

#### 调用数组方法

如果类数组就是任性的想用数组的方法怎么办呢？

既然无法直接调用，我们可以用 Function.call 间接调用。

```js
var arrayLike = {0: 'name', 1: 'age', 2: 'sex', length: 3 }

Array.prototype.join.call(arrayLike, '&'); // name&age&sex

Array.prototype.slice.call(arrayLike, 0); // ["name", "age", "sex"] // slice可以做到类数组转数组

Array.prototype.map.call(arrayLike, function(item){
  return item.toUpperCase();
});  // ["NAME", "AGE", "SEX"]
```

#### 类数组转数组

在上面的例子中已经提到了一种类数组转数组的方法，再补充三个：

```js
var arrayLike = {0: 'name', 1: 'age', 2: 'sex', length: 3 }
// 1. slice
Array.prototype.slice.call(arrayLike); // ["name", "age", "sex"] 
// 2. splice
Array.prototype.splice.call(arrayLike, 0); // ["name", "age", "sex"] 
// 3. ES6 Array.from
Array.from(arrayLike); // ["name", "age", "sex"] 
// 4. apply
Array.prototype.concat.apply([], arrayLike)
```

⚠️ **拓展**

[How does `Array.prototype.slice.call` work](https://stackoverflow.com/questions/7056925/how-does-array-prototype-slice-call-work)

>What happens under the hood is that when `.slice()` is called normally, `this` is an Array, and then it just iterates over that Array, and does its work.
>
>How is `this` in the `.slice()` function an Array? Because when you do:
>
>```js
>object.method();
>```
>
>...the `object` automatically becomes the value of `this` in the `method()`. So with:
>
>```js
>[1,2,3].slice()
>```
>
>...the `[1,2,3]` Array is set as the value of `this` in `.slice()`.
>
>------
>
>But what if you could substitute something else as the `this` value? As long as whatever you substitute has a numeric `.length` property, and a bunch of properties that are numeric indices, it should work. This type of object is often called an *array-like object*.
>
>The `.call()` and `.apply()` methods let you *manually* set the value of `this` in a function. So if we set the value of `this` in `.slice()` to an *array-like object*, `.slice()` w**ill just *assume* it's working with an Array**, and will do its thing.
>
>Take this plain object as an example.
>
>```js
>var my_object = {
>    '0': 'zero',
>    '1': 'one',
>    '2': 'two',
>    '3': 'three',
>    '4': 'four',
>    length: 5
>};
>```
>
>This is obviously not an Array, but if you can set it as the `this` value of `.slice()`, then it will just work, because it looks enough like an Array for `.slice()` to work properly.
>
>```js
>var sliced = Array.prototype.slice.call( my_object, 3 );
>```
>
>**Example:** http://jsfiddle.net/wSvkv/
>
>As you can see in the console, the result is what we expect:
>
>```js
>['three','four'];
>```
>
>**So this is what happens when you set an `arguments` object as the `this` value of `.slice()`. Because `arguments` has a `.length` property and a bunch of numeric indices, `.slice()` just goes about its work as if it were working on a real Array.**

#### Arguments 对象

Arguments 对象只定义在函数体中，包括了函数的参数和其他属性。在函数体中，arguments 指代该函数的 Arguments 对象。

举个例子：

```js
function foo(name, age, sex) { console.log(arguments); }
foo('name', 'age', 'sex')
```

<img src="https://camo.githubusercontent.com/993a101381ec9e9badf6591d841fd7deb53a7a8dde01bf17980cc2aefacc65d4/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f6d717971696e6766656e672f426c6f672f496d616765732f617267756d656e74732e706e67" alt="https://camo.githubusercontent.com/993a101381ec9e9badf6591d841fd7deb53a7a8dde01bf17980cc2aefacc65d4/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f6d717971696e6766656e672f426c6f672f496d616765732f617267756d656e74732e706e67" style="zoom: 67%;" />

我们可以看到除了类数组的 索引属性 和 length 属性之外，还有一个 callee 属性。

#### callee 属性

Arguments 对象的 callee 属性，通过它可以调用函数自身。讲个闭包经典面试题使用 callee 的解决方法：

```js
var data = [];
for (var i = 0; i < 3; i++) {
    (data[i] = function () {
       console.log(arguments.callee.i) 
    }).i = i;
}

data[0](); // 0
data[1](); // 1
data[2](); // 2
```

#### arguments 和对应参数的绑定

```js
function foo(name, age, sex, hobbit) {
    console.log(name, arguments[0]); // name name

    // 改变形参
    name = 'new name';
    console.log(name, arguments[0]); // new name new name

    // 改变arguments
    arguments[1] = 'new age';
    console.log(age, arguments[1]); // new age new age

    // 测试未传入的是否会绑定
    console.log(sex); // undefined
    sex = 'new sex';
    console.log(sex, arguments[2]); // new sex undefined

    arguments[3] = 'new hobbit';
    console.log(hobbit, arguments[3]); // undefined new hobbit
}

foo('name', 'age')
```

传入的参数，实参和 arguments 的值会共享，当没有传入时，实参与 arguments 值不会共享。除此之外，以上是在非严格模式下，如果是在严格模式下，实参和 arguments 是不会共享的。

#### arguments 应用

包括：

1. 参数不定长
2. 函数柯里化
3. 递归调用
4. 函数重载

摘自：[JavaScript深入之类数组对象与arguments](https://github.com/mqyqingfeng/Blog/issues/14)

