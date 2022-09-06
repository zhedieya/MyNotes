### Promise

#### 面试题：红绿灯问题

题目：红灯三秒亮一次，绿灯一秒亮一次，黄灯2秒亮一次；如何让三个灯不断交替重复亮灯？（用 Promse 实现）

三个亮灯函数已经存在：

```js
function red(){ console.log('red'); }
function green(){ console.log('green'); }
function yellow(){ console.log('yellow'); }
```

利用 then 和递归实现：

```js
// red green yellow 函数略
var light = function(timmer, cb){
    return new Promise(function(resolve, reject) {
        setTimeout(function() { cb(); resolve(); }, timmer);
    });
};

var step = function() {
    Promise.resolve().then(function(){ return light(3000, red); })
      .then(function(){ return light(2000, green); })
      .then(function(){ return light(1000, yellow); })
      .then(function(){ step(); }); // 注：递归执行
}
step();
```

#### promisify

有的时候，我们<font color=FF0000>需要将 callback 语法的 API 改造成 Promise 语法</font>，为此我们需要一个 promisify 的方法。

因为 callback 语法传参比较明确，最后一个参数传入回调函数，回调函数的第一个参数是一个错误信息，如果没有错误，就是 null，所以我们可以直接写出一个简单的 promisify 方法：

```js
function promisify(original) {
    return function (...args) {
        return new Promise((resolve, reject) => {
            args.push(function callback(err, ...values) {
                if (err) { return reject(err); }
                return resolve(...values)
            });
            original.call(this, ...args);
        });
    };
}
```

完整的可以参考 [es6-promisify](https://github.com/digitaldesignlabs/es6-promisify/blob/master/lib/promisify.js)

#### Promise 的局限性

##### 1. 错误被吃掉

首先我们要理解，什么是错误被吃掉，是指错误信息不被打印吗？并不是，举个例子：

```js
throw new Error('error');
console.log(233333);
```

在<mark>这种情况下，因为 throw error 的缘故，代码被阻断执行，并不会打印 233333</mark>，再举个例子：

```js
const promise = new Promise(null);
console.log(233333);
```

以上代码依然会被阻断执行，这是因为如果通过无效的方式使用 Promise，并且出现了一个错误阻碍了正常 Promise 的构造，结果会得到一个立刻跑出的异常，而不是一个被拒绝的 Promise。

然而再举个例子：

```js
let promise = new Promise(() => { throw new Error('error') });
console.log(2333333);
```

这次会正常的打印 `233333`，说明 <font color=FF0000>**Promise 内部的错误不会影响到 Promise 外部的代码**，而这种情况我们就通常称为 “吃掉错误”</font>。

其实这并不是 Promise 独有的局限性，try..catch 也是这样，同样会捕获一个异常并简单的吃掉错误。

而<font color=FF0000>**正是因为错误被吃掉，Promise 链中的错误很容易被忽略掉**，这也是为什么会一般推荐在 Promise 链的最后添加一个 catch 函数</font>，<font color=FF0000>因为对于一个没有错误处理函数的 Promise 链，**任何错误都会在链中被传播下去，直到你注册了错误处理函数**</font>。

##### 2. 单一值

<font color=FF0000>**Promise 只能有一个完成值或一个拒绝原因**</font>，然而<font color=FF0000>在真实使用的时候，往往需要传递多个值，一般做法都是构造一个对象或数组，然后再传递</font>，then 中获得这个值后，又会进行取值赋值的操作，每次封装和解封都无疑让代码变得笨重。

说真的，并没有什么好的方法，<font color=FF0000>建议使用 ES6 的解构赋值</font>：

```js
Promise.all([Promise.resolve(1), Promise.resolve(2)])
       .then(([x, y]) => { console.log(x, y); });
```

##### 3. 无法取消

<font color=FF0000>**Promise 一旦新建它就会立即执行，无法中途取消**</font>。

##### 4. 无法得知 pending 状态

<font color=FF0000>**当处于 pending 状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）**</font>。

摘自：[ES6 系列之我们来聊聊 Promise](https://github.com/mqyqingfeng/Blog/issues/30)