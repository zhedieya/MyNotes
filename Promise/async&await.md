[[理解 JavaScript 的 async/await]](https://segmentfault.com/a/1190000007535316)

#### async起什么作用

async 函数（包含函数语句、函数表达式、Lambda表达式）会返回一个 Promise 对象。如果在函数中`return`一个直接量，async会把这个直接量通过`Promise.resolve()`封装成 Promise 对象。

> `Promise.resolve(x)` 可以看作是 `new Promise(resolve => resolve(x))` 的简写，可以用于快速封装字面量对象或其他对象，将其封装成 Promise 实例。

如果 async 函数没有返回值，它会返回 `Promise.resolve(undefined)`。

Promise 的特点——无等待，所以在没有 `await` 的情况下执行 async 函数，它会立即执行，返回一个 Promise 对象，并且，绝不会阻塞后面的语句。这和普通返回Promise对象的函数并无二致。

#### await在等什么

await等待的是一个表达式，这个表达式返回Promise对象的处理结果。如果等待的不是Promise对象，则返回该值本身(也就是说没有特殊限制)

#### await等到后干嘛

await表达式的运算结果是它等待的东西

await表达式会暂停当前[async function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)的执行，等待Promise处理完成。若 Promise 正常处理(fulfilled)，其回调的resolve函数参数作为 await 表达式的值，继续执行async function;若 Promise 处理异常(rejected)，await 表达式会把 Promise 的异常原因抛出。

另外，如果 await 操作符后的表达式的值不是一个 Promise，则返回该值本身。

> 这就是 await 必须用在 async 函数中的原因。async 函数调用不会造成阻塞，它内部所有的阻塞都被封装在一个 Promise 对象中异步执行。

#### async function

`async` 声明一个函数是异步函数。异步函数会自动（由解释器）使用 Promise 封装返回值，同时，其内部可以使用 `await` 关键字。最终由 async/await/Promise 配合由解释器将类似同步方式编写的代码以正确的异步逻辑来调用

`async function`**它的接口符合异步函数的接口规范，即返回一个 `Promise` 对象，其内部可以使用 `await`**。但函数内部是如何实现的，这个声明管不了。内部可以用同步函数来实现，它就同步执行，只不过返回的是 Promise 封装的返回值（或封装的 `undefined`）；内部也可以用真正的异步操作来实现，比如setTimeout，Ajax 等，那它就是真异步；也就是说async只是声明了一个函数可以采用异步，但是具体是不是异步要看里面的具体实现。



### async await

ES2017 标准引入了 async 函数，使得异步操作变得更加方便。在异步处理上，async 函数就是 Generator 函数的语法糖。

```js
// 使用 generator
var fetch = require('node-fetch');
var co = require('co');

function* gen() {
    var r1 = yield fetch('https://api.github.com/users/github');
    var json1 = yield r1.json();
    console.log(json1.bio);
}
co(gen);
```

当你使用 async 时：

```js
// 使用 async
var fetch = require('node-fetch');

var fetchData = async function () {
    var r1 = await fetch('https://api.github.com/users/github');
    var json1 = await r1.json();
    console.log(json1.bio);
};
fetchData();
```

其实 <font color=FF0000>**async 函数的实现原理：就是将 Generator 函数 和 自动执行器，包装在一个函数里**</font>。

```js
async function fn(args) { ... }

// 等同于
function fn(args) {
  return spawn(function* () { ... });
}
```

<font color=FF0000>spawn 函数指的是 <font size=4>**自动执行器**</font>，就比如说 <font size=4>**co**</font></font>。再加上 <font color=FF0000 size=4>**async 函数返回一个 Promise 对象**</font>，你也可以理解为 async 函数是基于 Promise 和 Generator 的一层封装。

#### async 与 Promise

严谨的说：<font color=FF0000>async 是一种语法，Promise 是一个内置对象，两者并不具备可比性</font>；更何况 async 函数也返回一个 Promise 对象……

这里主要是展示一些场景，使用 async 会比使用 Promise 更优雅的处理异步流程。

##### 1. 代码更加简洁（注：下面的每个示例都是原先写法 和 使用 async 的写法比较）

```js
/* 示例一 */
function fetch() {
  return ( fetchData().then( () => { return "done" } ); )
}

async function fetch() {
  await fetchData()
  return "done"
};
```

```js
/* 示例二 */
function fetch() {
  return fetchData().then(data => {
    if (data.moreData) {
        return fetchAnotherData(data).then( moreData => { return moreData } )
    } else { return data }
  });
}

async function fetch() {
  const data = await fetchData()
  if (data.moreData) {
    const moreData = await fetchAnotherData(data);
    return moreData
  } else { return data }
};
```

```js
/* 示例三 */
function fetch() {
  return (
    fetchData().then(value1 => { return fetchMoreData(value1) })
    					 .then(value2 => { return fetchMoreData2(value2) })
  )
}

async function fetch() {
  const value1 = await fetchData()
  const value2 = await fetchMoreData(value1)
  return fetchMoreData2(value2)
};
```

##### 2. 错误处理

```js
function fetch() {
  try {
    fetchData()
      .then(result => { const data = JSON.parse(result) })
      .catch((err) => { console.log(err) })
  } catch (err) { console.log(err) }
}
```

在这段代码中，try / catch 能捕获 fetchData() 中的一些 Promise 构造错误，但是不能捕获 JSON.parse 抛出的异常，如果要处理 JSON.parse 抛出的异常，需要添加 catch 函数重复一遍异常处理的逻辑。在实际项目中，错误处理逻辑可能会很复杂，这会导致冗余的代码。

```js
async function fetch() {
  try { const data = JSON.parse(await fetchData()) }
  catch (err) { console.log(err) }
};
```

async / await 的出现使得 try/catch 就可以捕获同步和异步的错误。**注：**关于 async / await 可以使用 [await-to-js](https://github.com/scopsy/await-to-js) 更优雅地捕获异常

##### 3. 调试

```js
const fetchData = () => new Promise((resolve) => setTimeout(resolve, 1000, 1))
const fetchMoreData = (value) => new Promise((resolve) => setTimeout(resolve, 1000, value + 1))
const fetchMoreData2 = (value) => new Promise((resolve) => setTimeout(resolve, 1000, value + 2))

function fetch() {
  return (
    fetchData()
    .then((value1) => { console.log(value1) return fetchMoreData(value1) })
    .then(value2 => { return fetchMoreData2(value2) })
  )
}
const res = fetch();
console.log(res);
```

![https://camo.githubusercontent.com/38c17c920b970173d0b8ba41f26edf9e41cefdf9db6d4c7466333b6b137e1eef/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f6d717971696e6766656e672f426c6f672f496d616765732f4553362f6173796e632f70726f6d6973652e676966](https://s2.loli.net/2022/04/19/MQWNvKT1n9Ryawh.gif)

因为 then 中的代码是异步执行，所以当你打断点的时候，代码不会顺序执行，尤其当你使用 step over 的时候，then 函数会直接进入下一个 then 函数。

```js
const fetchData = () => new Promise((resolve) => setTimeout(resolve, 1000, 1))
const fetchMoreData = () => new Promise((resolve) => setTimeout(resolve, 1000, 2))
const fetchMoreData2 = () => new Promise((resolve) => setTimeout(resolve, 1000, 3))

async function fetch() {
  const value1 = await fetchData()
  const value2 = await fetchMoreData(value1)
  return fetchMoreData2(value2)
};
const res = fetch();
console.log(res);
```

![https://camo.githubusercontent.com/8348dc27d42ca5eff110cbe13a86979871721da5cca45fe99793acf4cd23450a/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f6d717971696e6766656e672f426c6f672f496d616765732f4553362f6173796e632f6173796e632e676966](https://s2.loli.net/2022/04/19/zsL9Ud483SjKkPr.gif)

而使用 async 的时候，则可以像调试同步代码一样调试。

#### async 地狱

async 地狱主要是<mark>指开发者贪图语法上的简洁而让原本可以并行执行的内容变成了顺序执行，从而影响了性能</mark>，但用地狱形容有点夸张了

##### 例子一

举个例子：

```js
(async () => {
  const getList = await getList();
  const getAnotherList = await getAnotherList();
})();
```

`getList()` 和 `getAnotherList()` 其实并没有依赖关系，但是现在的这种写法，虽然简洁，却导致了 `getAnotherList()` 只能在 `getList()` 返回后才会执行，从而导致了多一倍的请求时间。为了解决这个问题，我们可以改成这样：

```js
(async () => {
  const listPromise = getList();
  const anotherListPromise = getAnotherList();
  await listPromise;
  await anotherListPromise;
})();
```

也可以使用 Promise.all()：

```js
(async () => {
  Promise.all([getList(), getAnotherList()]).then(...);
})();
```

##### 例子二

当然上面这个例子比较简单，我们再来扩充一下：

```js
(async () => {
  const listPromise = await getList();
  const anotherListPromise = await getAnotherList();

  // do something

  await submit(listData);
  await submit(anotherListData);
})();
```

因为 await 的特性，整个例子有明显的先后顺序，然而 `getList()` 和 `getAnotherList()` 其实并无依赖，`submit(listData)` 和 `submit(anotherListData)` 也没有依赖关系，那么对于这种例子，我们该怎么改写呢？

**基本分为三个步骤：**

1. **找出依赖关系：**在这里，submit(listData) 需要在 getList() 之后，submit(anotherListData) 需要在 anotherListPromise() 之后。

2. **将互相依赖的语句包裹在 async 函数中**

   ```js
   async function handleList() {
     const listPromise = await getList();
     // ...
     await submit(listData);
   }
   
   async function handleAnotherList() {
     const anotherListPromise = await getAnotherList()
     // ...
     await submit(anotherListData)
   }
   ```

3. **并发执行 async 函数**

   ```js
   async function handleList() {
     const listPromise = await getList();
     // ...
     await submit(listData);
   }
   
   async function handleAnotherList() {
     const anotherListPromise = await getAnotherList()
     // ...
     await submit(anotherListData)
   }
   
   // 方法一
   (async () => {
     const handleListPromise = handleList()
     const handleAnotherListPromise = handleAnotherList()
     await handleListPromise
     await handleAnotherListPromise
   })()
   
   // 方法二
   (async () => {
     Promise.all([handleList(), handleAnotherList()]).then()
   })()
   ```

#### 继发与并发

##### 问题：给定一个 URL 数组，如何实现接口的 继发 和 并发 ？

**async 继发实现：**

```js
// 继发一
async function loadData() {
  var res1 = await fetch(url1);
  var res2 = await fetch(url2);
  var res3 = await fetch(url3);
  return "whew all done";
}
// 继发二
async function loadData(urls) {
  for (const url of urls) {
    const response = await fetch(url);
    console.log(await response.text());
  }
}
```

**async 并发实现：**

```js
// 并发一
async function loadData() {
  var res = await Promise.all([fetch(url1), fetch(url2), fetch(url3)]);
  return "whew all done";
}
// 并发二
async function loadData(urls) {
  // 并发读取 url
  const textPromises = urls.map(async url => {
    const response = await fetch(url);
    return response.text();
  });
  // 按次序输出
  for (const textPromise of textPromises) {
    console.log(await textPromise);
  }
}
```

#### async 错误捕获

尽管我们可以使用 try catch 捕获错误，但是当我们需要捕获多个错误并做不同的处理时，很快 try catch 就会导致代码杂乱，就比如：

```js
async function asyncTask(cb) {
    try {
       const user = await UserModel.findById(1);
       if(!user) return cb('No user found');
    } catch(e) {
        return cb('Unexpected error occurred');
    }

    try {
       const savedTask = await TaskModel({userId: user.id, name: 'Demo Task'});
    } catch(e) {
        return cb('Error occurred while saving task');
    }

    if(user.notificationsEnabled) {
        try {
            await NotificationService.sendNotification(user.id, 'Task Created');
        } catch(e) {
            return cb('Error while sending notification');
        }
    }

    if(savedTask.assignedUser.id !== user.id) {
        try {
            await NotificationService.sendNotification(savedTask.assignedUser.id, 'Task was created for you');
        } catch(e) {
            return cb('Error while sending notification');
        }
    }

    cb(null, savedTask);
}
```

为了简化这种错误的捕获，我们可以给 await 后的 promise 对象添加 catch 函数，为此我们需要写一个 helper：

```js
// to.js
export default function to(promise) {
   return promise.then(data => {
      return [null, data];
   })
   .catch(err => [err]);
}
```

整个错误捕获的代码可以简化为：

```js
import to from './to.js';

async function asyncTask() {
     let err, user, savedTask;

     [err, user] = await to(UserModel.findById(1));
     if(!user) throw new CustomerError('No user found');

     [err, savedTask] = await to(TaskModel({userId: user.id, name: 'Demo Task'}));
     if(err) throw new CustomError('Error occurred while saving task');

    if(user.notificationsEnabled) {
       const [err] = await to(NotificationService.sendNotification(user.id, 'Task Created'));
       if (err) console.error('Just log the error and continue flow');
    }
}
```

#### async 的一些讨论

##### async 会取代 Generator 吗？

Generator 本来是用作生成器，使用 Generator 处理异步请求只是一个比较 hack 的用法。在异步方面，async 可以取代 Generator，但是 <font color=FF0000>async 和 Generator 两个语法本身是用来解决不同的问题的</font>。

##### async 会取代 Promise 吗？

1. async 函数返回一个 Promise 对象
2. 面对复杂的异步流程，Promise 提供的 all 和 race 会更加好用
3. Promise 本身是一个对象，所以可以在代码中任意传递
4. async 的支持率还很低，即使有 Babel，编译后也要增加 1000 行左右。

摘自：[ES6 系列之我们来聊聊 Async](https://github.com/mqyqingfeng/Blog/issues/100)

#### 《 JavaScript 高级程序设计》第四版 中的 async await

async 关键字用于声明异步函数。另外，async 也可以用在类定义中

```js
class Foo {
  async foo() {}
}
```

<font color=FF0000>使用 async 关键字可以让函数具有异步特征，但总体上其代码仍然是同步求值的</font>。而在参数或闭包方面，异步函数仍然具有普通 JavaScript 函数的正常行为。正如下面的例子所示，foo() 函数仍然会在后面的指令之前被求值：

```js
async function foo() { console.log(1); }
foo();
console.log(2);
// 1, 2
```

不过，<font color=FF0000>如果 异步函数使用 return 关键字返回了值（ 如果没有 return 则会返回 undefined ），这 个值会被 Promise.resolve() 包装成一个 Promise 对象</font>。异步函数 始终返回 Promise 对象。在函数外部调用这个函数可以得到它返回的 Promise ：

```js
async function foo() {
  console.log(1);
  return 3;
}
// 给返回的 Promise 添加一个解决处理程序
foo().then(console.log);
console.log(2);
// 1, 2, 3
```

当然，直接返回一个 Promise 对象也是一样的：

```js
async function foo() {
  console.log(1);
  return Promise.resolve(3);
}
// 给返回的 Promise 添加一个解决处理程序
foo().then(console.log);
console.log(2);
// 1, 2, 3
```

异步函数的返回值期待（ 但实际上并不要求 ）一个实现 thenable 接口的对象，但常规的值也可以。 如果返回的是实现 thenable 接口的对象，则这个对象可以由提供给 then() 的处理程序 “解包”（**注：**即 .then(fn) 中的函数（即这里的 fn）可以对 Promise 进行解包，获取 Promise 的内容）。如果不是，则返回值就被当作 resolved 的 Promise。

```js
// 返回一个原始值
async function foo() {  return 'foo'; }
foo().then(val => console.log(val)); // foo。注：如果 typeof val 结果是 string

// 返回一个实现了 thenable 接口的非 Promise 对象
async function baz() {
  const thenable = { 
    then(callback) { callback('baz'); }
  }; 
  return thenable;
}
baz().then(console.log); // baz

// 返回一个 Promise
async function qux() {
  return Promise.resolve('qux');
}
qux().then(console.log); // qux
```

与在 Promise 处理程序中一样，在异步函数中 抛出错误 会返回拒绝的 Promise：

```js
async function foo() {
  console.log(1);
  throw 3; 
}
// 给返回的 Promise 添加一个拒绝处理程序
foo().catch(console.log); // 注：这里 foo() 返回的，依然是一个 Promise
console.log(2);
// 1, 2, 3
```

不过，rejected 的 Promise 的错误不会被异步函数捕获：

```js
async function foo() {
  console.log(1);
  Promise.reject(3);
}
// Attach a rejected handler to the returned promise
foo().catch(console.log);
console.log(2);
// 1, 2, Uncaught (in promise): 3
```

##### await

await 关键字期待（ 但实际上并不要求 ）一个实现 thenable 接口的对象，但常规的值也可以。如果是实现 thenable 接口的对象，则<font color=FF0000>这个对象可以由 await 来 “解包”</font>（**注：**解包 Promise ）。如果不是，则这个值就被当作 resolved 的 Promise

```js
// 等待一个原始值
async function foo() { console.log( await 'foo' ); }
foo(); // foo

// 等待一个实现了 thenable 接口的非 Promise 对象
async function baz() { 
  const thenable = {
    then(callback) { callback('baz'); }
  };
  console.log(await thenable);
}
baz(); // baz

// 等待一个 Promise
async function qux() {
  console.log(await Promise.resolve('qux'));
}
qux(); // qux
```

等待会抛出错误的同步操作，会返回 rejected 的 Promise：

```js
async function foo() {
  console.log(1);
  await (() => { throw 3; })(); 
}
// 给返回的 Promise 添加一个拒绝处理程序
foo().catch(console.log);
console.log(2);
// 1, 2, 3
```

如前面的例子所示，<font color=FF0000>单独的 Promise.reject() 不会被异步函数捕获，而会抛出未捕获错误</font>。不过，对 rejected 的 Promise 使用 await 则会释放 ( unwrap ) 错误值（将 rejected 的 Promise 返回 ）：

```js
async function foo() {
  console.log(1);
  await Promise.reject(3);
  console.log(4); // 这行代码不会执行
}
// 给返回的 Promise 添加一个拒绝处理程序
foo().catch(console.log);
console.log(2);
// 1, 2, 3
```

JavaScript 运行时在碰 到 await 关键字时，会记录在哪里暂停执行。等到 await 右边的值可用了，JavaScript 运行时会向消息队列中推送一个任务，这个任务会恢复异步函数的执行。

另外，TC39 对 await 做了修改，下例中 await Promise.resolve('foo') <font color=FF0000>只会生成一个异步任务</font>，和 await 'bar' 一样；所以先于后者执行。

```js
async function foo() { console.log(await Promise.resolve('foo')); }
async function bar() { console.log(await 'bar'); }
async function baz() { console.log('baz'); }
foo();
bar();
baz();
// baz, foo, bar
```

摘自：《 JavaScript 高级程序设计》第四版 §11.3.1 P348







