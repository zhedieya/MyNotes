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











