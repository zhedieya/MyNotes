[toc]

### JS类型判断

#### typeof

typeof 除了是一个函数外，也是一个<font color=FF0000>运算符</font>。引用《JavaScript 权威指南》中对 typeof 的介绍：

> typeof 是一元操作符，放在其单个操作数的前面，操作数可以是任意类型。返回值为表示操作数类型的一个字符串。

在 ES6 前，JavaScript 共六种数据类型，分别是：Undefined、Null、Boolean、Number、String、Object

然而当我们使用 typeof 对这些数据类型的值进行操作的时候，返回的结果却不是一一对应，分别是：undefined、object、boolean、number、string、object。注意以上都是小写的字符串。Null 和 Object 类型都返回了 object 字符串。

尽管不能一一对应，但是 typeof 却能检测出 函数类型 function。

所以 <font color=FF0000>typeof 能检测出六种类型的值</font>（**注：**下面的 [[#typeof 新的可识别类型的补充]] 有补充：Symbol 和 BigInt 两种情况也可以识别其类型），但是，除此之外 Object 下还有很多细分的类型，如 Array、Function、Date、RegExp、Error 等。而对这些类型使用 typeof 操作符，返回的都是 object；无法良好的区分它们。这时候可以使用 Object.prototype.toString.call() 获取 '\[object RefObjectType]'；另外，可以通过正则表达式获取 refObjectType（获取类型代码，学习自：[你最少用几行代码实现深拷贝？](https://juejin.cn/post/7075351322014253064) ）：

```js
function getType(obj) {
    return Object.prototype.toString.call(obj).replaceAll(new RegExp(/\[|\]|object /g), "");
}

// 使用
getType(Promise.resolve('foo')) // Promise
```

##### typeof 新的可识别类型的补充

现在 <font color=FF0000>**不完全一样**</font> 了，至少对于 symbol、bigint 是可以判别的；至于其他的（比如 Date ）就不行了。如下摘自：[MDN - typeof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof)

> | 型                                              | 结果             |
> | :---------------------------------------------- | :--------------- |
> | Undefined                                       | "undefined"      |
> | Null                                            | "object"         |
> | Boolean                                         | "boolean"        |
> | Number                                          | "number"         |
> | BigInt(ECMAScript 2020 新增)                    | "bigint"         |
> | String                                          | "string"         |
> | Symbol (ECMAScript 2015 新增)                   | "symbol"         |
> | 宿主对象（由 JS 环境提供）                      | *取决于具体实现* |
> | Function 对象 (按照 ECMA-262 规范实现 [[Call]]) | "function"       |
> | 其他任何对象                                    | "object"         |

#### Object.prototype.toString

先献上 ES5 规范地址：https://es5.github.io/#x15.2.4.2。

在第 15.2.4.2 节讲的就是 Object.prototype.toString()，为了不误导大家，我先奉上英文版：

> When the toString method is called, the following steps are taken:

> 1. If the **this** value is **undefined**, return "**[object Undefined]**".
> 2. If the **this** value is **null**, return "**[object Null]**".
> 3. Let *O* be the result of calling ToObject passing the **this** value as the argument.
> 4. Let *class* be the value of the [[Class]] internal property of *O*.
> 5. Return the String value that is the result of concatenating the three Strings "**[object** ", *class*, and "**]**".

凡是规范上加粗或者斜体的，在这里我也加粗或者斜体了，就是要让大家感受原汁原味的规范！

**如果没有看懂，就不妨看看我理解的：**

当 toString 方法被调用的时候，下面的步骤会被执行：

1. 如果 this 值是 undefined，就返回 [object Undefined]

2. 如果 this 的值是 null，就返回 [object Null]

   **注：**上面是两种异常情况 undefined 和 null 的处理

3. <font color=FF0000>让 O 成为 ToObject(this) 的结果</font>

4. <font color=FF0000>让 class 成为 O 的内部属性 [[Class]] 的值</font>

5. 最后返回由 "[object " 和 class 和 "]" 三个部分组成的字符串

通过规范，我们至少知道了调用 Object.prototype.toString 会返回一个由 "[object " 和 class 和 "]" 组成的字符串，而 class 是要判断的对象的内部属性。

经过实验（代码略，见原文），Object.prototyep.toString.call() 可以识别 Number、String、Boolean、Undefined、 Null、Object、Array、Date、Error、EegExp、Function 这 11 种类型之外，还可以识别 Math、JSON、arguments

```js
console.log(Object.prototype.toString.call(Math)); // [object Math]
console.log(Object.prototype.toString.call(JSON)); // [object JSON]
```

所以我们可以识别至少 14 种类型，当然我们也可以算出来，[[class]] 属性至少有 12 个。

**注：**还有上面提及的 Symbol 和 BigInt。另外，还有 Promise、Set、Map、weakSet、weakMap、location、history、window（只在浏览器下生效，结果为 '[object Window]' ）、 globalThis（ globalThis 在不同宿主环境的值不同，浏览器下的值为 '[object Window]'，在 Node 中为 '[object global]' ）

#### 实现 type 方法：

需要注意的是<font color=FF0000>边界情况</font>：在 IE6 中，null 和 undefined 会被 Object.prototype.toString 识别成 [object Object]

```js
var class2type = {};

// 生成class2type映射
"Boolean Number String Function Array Date RegExp Object Error".split(" ").map(function(item, index) {
    class2type["[object " + item + "]"] = item.toLowerCase();
})

function type(obj) {
    // 一箭双雕，因为：(undefined == null) === true
    if (obj == null) {
        return obj + "";
    }
    return typeof obj === "object" || typeof obj === "function"
      		 ? class2type[Object.prototype.toString.call(obj)] || "object"
    			 : typeof obj;
}
```

#### 判断类型有四种

1. **typeof**

2. **constructor**

   使用  ***.constructor.name** 可以实现判断类型功能（**注：**泛用性没有 toString 好，比如 null、undefined 不可用，arguments 的结果还是 Object。另外，这里的原理是： constructor 是一个函数，Function 有 name 这个属性 ），示例如下：

   ```js
   const reg = /abc/
   console.log(Object.getPrototypeOf(reg).constructor.name) // 'RegExp'
   ```

   另外，在看 [「2021」高频前端面试题汇总之CSS篇](https://juejin.cn/post/6905539198107942919) 时，发现这样一个之前没见过的写法（虽然没什么）：

   ```js
   console.log((2).constructor === Number); // true
   ```

   其他的基础类型也可以用，这就等价于：

   ```js
   const testNum = 2
   console.log(testNum.constructor === Number); // true
   ```

3. **instanceof**

4. **Object.prototype.toString**

#### 开发中的其他情况

上面编写了 type 函数，可以检测出常见的数据类型；然而在开发中还有更加复杂的判断，比如 plainObject、空对象、Window 对象等

##### plainObject

plainObject 来自于 jQuery，可以翻译成<mark>纯粹的对象，所谓“纯粹的对象”</mark>，就是<font color=FF0000>该对象是 **通过 "{}" 或 "new Object" 创建的**，该对象含有零个或者多个键值对</font>。

之所以要判断是不是 plainObject，是为了跟其他的 JavaScript对象如 null，数组，宿主对象（documents）等作区分，因为这些用 typeof 都会返回object。

jQuery 提供了 isPlainObject 方法进行判断，先让我们看看使用的效果：

```js
function Person(name) {
    this.name = name;
}

console.log( $.isPlainObject( {} ))                              // true
console.log( $.isPlainObject( new Object ) )                     // true
console.log( $.isPlainObject( Object.create(null) ) )            // true
console.log( $.isPlainObject( Object.assign({a: 1}, {b:  2}) ))  // true
console.log( $.isPlainObject( new Person('yayu') ))              // false
console.log( $.isPlainObject( Object.create({}) ))               // false
```

由此我们可以看到，除了 {} 和 new Object 创建的之外，jQuery 认为 <font color=FF0000>**一个没有原型的对象也是一个纯粹的对象**</font>。

源码实现，有些复杂，略。

##### EmptyObject

jQuery 提供了 isEmptyObject 方法来<font color=FF0000>判断是否是空对象</font>，代码简单，我们直接看源码：

```js
function isEmptyObject( obj ) {
    var name;
    for ( name in obj ) {
        return false;
    }
    return true;
}
```

其实所谓的 isEmptyObject 就是判断是否有属性，<font color=FF0000>for 循环一旦执行，就说明有属性，有属性就会返回 false</font>。

但是根据这个源码我们可以看出 isEmptyObject 实际上判断的并不仅仅是空对象。举个栗子：

```js
console.log( $.isEmptyObject({}) );        // true
console.log( $.isEmptyObject([]) );        // true
console.log( $.isEmptyObject(null) );      // true
console.log( $.isEmptyObject(undefin ed)); // true
console.log( $.isEmptyObject(1) );         // true
console.log( $.isEmptyObject('') );        // true
console.log( $.isEmptyObject(true) );      // true
```

以上都会返回 true。

但是既然 jQuery 是这样写，可能是因为考虑到实际开发中 isEmptyObject 用来判断 {} 和 {a: 1} 是足够的吧。如果真的是只判断 {}，完全可以结合上面写的 type 函数筛选掉不适合的情况。

##### Window 对象

Window 对象作为客户端 JavaScript 的全局对象，它有一个 window 属性指向自身，这点在[《JavaScript深入之变量对象》](https://github.com/mqyqingfeng/Blog/issues/5)中讲到过。我们可以利用这个特性判断是否是 Window 对象。

```js
function isWindow( obj ) {
    return obj != null && obj === obj.window;
}
```

##### isArrayLike

isArrayLike，看名字可能会让我们觉得这是判断类数组对象的，其实不仅仅是这样，<font color=FF0000>jQuery 实现的 isArrayLike，数组和类数组都会返回 true</font>。因为源码比较简单，我们直接看源码：

```js
function isArrayLike(obj) {
  
    // obj 必须有 length属性
    var length = !!obj && "length" in obj && obj.length;
    var typeRes = type(obj);

    // 排除掉函数和 Window 对象
    if (typeRes === "function" || isWindow(obj)) {
        return false;
    }

    return typeRes === "array" || length === 0 ||
        typeof length === "number" && length > 0 && (length - 1) in obj;
}
```

重点分析 return 这一行，使用了或语句，只要一个为 true，结果就返回 true。

**所以如果 isArrayLike 返回 true，至少要满足三个条件之一：**

1. 是数组
2. <font color=FF0000>长度为 0</font>
3. <font color=FF0000>length 属性是大于 0 的数字类型，并且 obj[length - 1] 必须存在</font>

第一个就不说了，看<font color=FF0000>**第二个条件**</font>，为什么长度为 0 就可以直接判断为 true 呢？那我们写个对象：

```js
var obj = {a: 1, b: 2, length: 0}
```

isArrayLike 函数就会返回 true，那这个合理吗？回答合不合理之前，我们先看一个例子：

```js
function a(){
    console.log(isArrayLike(arguments))
}
a();
```

如果我们去掉 length === 0 这个判断，就会打印 false，然而我们都知道 <font color=FF0000>arguments 是一个类数组对象，这里是应该返回 true 的</font>。

所以<font color=FF0000>是不是为了 **放过 “空的 arguments”** 时也放过了一些存在争议的对象呢</font>？

<font color=FF0000>**第三个条件**</font>：length 是数字，并且 length > 0 且 最后一个元素存在。为什么仅仅要求最后一个元素存在呢？

让我们先想下数组是不是可以这样写：

```js
var arr = [,,3]
```

当我们写一个对应的类数组对象就是：

```js
var arrLike = {
    2: 3,
    length: 3
}
```

也就是说当我们<font color=FF0000>**在数组中用逗号直接跳过的时候，我们认为该元素是不存在的**</font>，<font color=FF0000>类数组对象中也就不用写这个元素，但是 **最后一个元素是一定要写的，要不然 length 的长度就不会是最后一个元素的 key 值加 1**</font>。比如数组可以这样写：

```js
var arr = [1,,];
console.log(arr.length) // 2
```

但是类数组对象就只能写成：

```js
var arrLike = {
    0: 1,
    length: 1
}
```

所以符合条件的类数组对象是一定存在最后一个元素的！

##### isElement

isElement 判断是不是 DOM 元素。

```js
isElement = function(obj) {
    return !!(obj && obj.nodeType === 1);
};
```

**注：**只读属性 Node.nodeType 表示的是该节点的类型。详见：[MDN - Node.nodeType](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/nodeType)

摘自：[JavaScript专题之类型判断(上)  ](https://github.com/mqyqingfeng/Blog/issues/28) 和  [JavaScript专题之类型判断(下)](