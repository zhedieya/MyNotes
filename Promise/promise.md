[toc]

https://blog.csdn.net/weixin_44972008/article/details/113779708

### 一.预备知识

### 二.Promise的理解和使用

#### 2.1 Promise是什么

##### 1.理解Promise

- 抽象表达：Promise是JS中进行异步编程的新的解决方案(旧方案是单纯使用回调函数)

>  推荐阅读：https://blog.csdn.net/weixin_44972008/article/details/114380206

   异步编程： ①fs 文件操作 ②数据库操作 ③Ajax ④定时器

- 具体表达：
  
  ①从语法上看：Promise是一个构造函数 (自己身上有all、reject、resolve这几个方法，原型上有then、catch等方法)

  ②从功能上看：Promise对象用来封装一个异步操作并可以获取其成功/失败的结果值
  
- 阮一峰的解释：
  
  所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果
  
  从语法上说，Promise 是一个对象，从它可以获取异步操作的消息
  
  Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理

##### 2.Promise状态

也就是实例对象promise中的一个属性 `PromiseState`.

1.`pending` 变为 `resolved/fullfilled`

2.`pending` 变为 `rejected`

注意:

- 对象的状态不受外界影响
- 只有这两种，且一个 promise 对象只能改变一次
- 一旦状态改变，就不会再变，任何时候都可以得到这个结果
- 无论成功还是失败，都会有一个结果数据。成功的结果数据一般称为 value，而失败的一般称为 reason。

##### 3.Promise对象的值

也就是实例对象promise的另一个属性`PromiseResult`

保存着对象「成功/失败」的结果`value reason`

`resolve`/`reject`可以修改值

##### 4.Promise的基本流程

![image-20220806205601006](https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20220806205601006.png)

##### 5.Promise的基本使用

```javascript
const promise = new Promise(function(resolve, reject) {
  // ... some code
  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(reason);
  }
});
```

`Promise`构造函数接受**一个函数**（执行器函数）作为参数，该函数的**两个参数**分别是`resolve`和`reject`。它们是**两个函数**，由 JavaScript 引擎提供，不用自己部署。

`resolve`函数的作用是，将`Promise`对象的状态从“未完成”变为“成功”（即从`pending`变为`resolved`），在**异步操作成功**时调用，并将异步操作的结果，作为参

数`value`传递出去；

`reject`函数的作用是，将`Promise`对象的状态从“未完成”变为“失败”（即从`pending`变为`rejected`，在**异步操作失败**时调用，并将异步操作报出的错误，作为

参数`error`/`reason`传递出去。

`Promise`实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数。

```javascript
promise.then(function(value) {
  // success
}, function(reason) {
  // failure
});
```

`then`方法可以接受**两个回调函数**作为参数。

第一个回调函数`onResolved()`是`Promise`对象的状态变为`resolved`时调用

第二个回调函数`onRejected()`是`Promise`对象的状态变为`rejected`时调用

这两个函数都是可选的，不一定要提供。它们都接受`Promise`对象传出的值作为参数

- 🌰

  ```javascript
  // 创建一个新的p对象promise
  const p = new Promise((resolve, reject) => { // 执行器函数
    // 执行异步操作任务
    setTimeout(() => {
      const time = Date.now() 
      // 如果当前时间是偶数代表成功，否则失败
      if (time % 2 == 0) {
        // 如果成功，调用resolve(value)
        resolve('成功的数据，time=' + time)
      } else {
        // 如果失败，调用reject(reason)
        reject('失败的数据，time=' + time)
      }
    }, 1000);
  })
  
  p.then(
    value => { // 接收得到成功的value数据 onResolved
      console.log('成功的回调', value)  // 成功的回调 成功的数据，time=1615015043258
    },
    reason => { // 接收得到失败的reason数据 onRejected
      console.log('失败的回调', reason)    // 失败的回调 失败的数据，time=1615014995315
    }
  )
  ```

- > .then() 和执行器(executor)同步执行，.then()中的回调函数异步执行

#### 2.2 为什么要用Promise

##### 1.指定回调函数的方式更加灵活

- 旧的：必须在启动异步任务前指定

  ```javascript
  // 1. 纯回调的形式
  // 成功的回调函数
  function successCallback(result) {
    console.log("声音文件创建成功：" + result);
  }
  // 失败的回调函数
  function failureCallback(error) {
    console.log("声音文件创建失败：" + error);
  }
  // 必须先指定回调函数，再执行异步任务
  createAudioFileAsync(audioSettings, successCallback, failureCallback) // 回调函数在执行异步任务（函数）前就要指定
  ```

- **promise：**启动异步任务=>返回promise对象=>给promise对象绑定回调函数(甚至可以在异步任务结束后指定)

  ```javascript
  // 2. 使用Promise
  const promise = createAudioFileAsync(audioSettings);  // 执行2秒
  setTimeout(() => {
    promise.then(successCallback, failureCallback) // 也可以获取
  }, 3000);
  ```

  

##### 2.支持链式调用，可以解决回调地狱问题

什么是回调地狱？

> 回调函数嵌套使用，外部回调函数异步执行的结果是嵌套的回调函数执行的条件

回调地狱问题：

```javascript
doSomething(function(result) {
  doSomethingElse(result, function(newResult) {
    doThirdThing(newResult, function(finalResult) {
      console.log('Got the final result:' + finalResult)
    }, failureCallback)
  }, failureCallback)
}, failureCallback)
```

回调地狱缺点：

1. 不便于阅读
2. 不便于异常处理

**解决方案：**

- promise链式调用

  

#### 2.3 如何使用Promise

##### 1. Promise 构造函数：`Promise(executor) {}`

- `executor`函数：**同步执行**`(resolve, reject) => {}`
- `resolve` 函数：内部定义成功时调用的函数 `resolve(value)`
- `reject`  函数：内部定义失败时调用的函数 `reject(reason)`

说明：`executor`是执行器，会在`Promise`内部立即同步回调，异步操作`resolve`/`reject`就在`executor`中执行

##### 2.Promise.prototype.then 方法：`p.then(onResolved, onRejected)`
指定两个回调（成功+失败）

- `onResolved` 函数：成功的回调函数 `(value) => {}`
- `onRejected` 函数：失败的回调函数 `(reason) => {}`

说明：指定用于得到成功`value`的成功回调和用于得到失败`reason`的失败回调，返回一个新的`promise`对象

##### 3.Promise.prototype.catch 方法：`p.catch(onRejected)`
指定失败的回调

- `onRejected`函数：失败的回调函数`(reason) => {}`

说明：这是`then()`的语法糖，相当于`then(undefined, onRejected)`

```javascript
new Promise((resolve, reject) => { // excutor执行器函数
 setTimeout(() => {
   if(...) {
     resolve('成功的数据') // resolve()函数
   } else { 
     reject('失败的数据') //reject()函数
    }
 }, 1000)
}).then(
 value => { // onResolved()函数
  console.log(value) // 成功的数据
}
).catch(
 reason => { // onRejected()函数
  console.log(reason) // 失败的数据
}
)
```

##### 4.Promise.resolve 方法：`Promise.resolve(value)`

`value`：将被`Promise`对象解析的参数，也可以是一个成功或失败的`Promise`对象

- 返回：返回一个带着给定值解析过的 Promise 对象，如果参数本身就是一个 Promise 对象，则直接返回这个 Promise 对象。

1.如果传入的参数为非Promise类型的对象, 则返回的结果为成功promise对象

```javascript
let p1 = Promise.resolve(521);
console.log(p1); // Promise {<fulfilled>: 521}
```

2.如果传入的参数为 Promise 对象, 则参数的结果决定了 resolve 的结果

```javascript
let p2 = Promise.resolve(new Promise((resolve, reject) => {
    // resolve('OK'); // 成功的Promise
    reject('Error');
}));
console.log(p2);
p2.catch(reason => {
    console.log(reason);
})
```

##### 5.Promise.reject 方法：`Promise.reject(reason)`

`reason`：失败的原因

说明：返回一个失败的 `promise` 对象

```javascript
let p = Promise.reject(521);
let p2 = Promise.reject('iloveyou');
let p3 = Promise.reject(new Promise((resolve, reject) => {
    resolve('OK');
}));

console.log(p);
console.log(p2);
console.log(p3);
```

<img src="https://img-blog.csdnimg.cn/20210226213630534.png" alt="20210226213630534" style="zoom:90%;" align="left" />

- Promise.resolve()/Promise.reject() 方法就是一个语法糖
- 用来快速得到Promise对象

```javascript
//产生一个成功值为1的promise对象
new Promise((resolve, reject) => {
 resolve(1)
})
//相当于
const p1 = Promise.resolve(1)
const p2 = Promise.resolve(2)
const p3 = Promise.reject(3)

p1.then(value => {console.log(value)}) // 1
p2.then(value => {console.log(value)}) // 2
p3.catch(reason => {console.log(reason)}) // 3
```

##### 6.Promise.all 方法`Promise.all(iterable)`

iterable：包含 n 个 `promise`的可迭代对象，如`Array`或`String`

说明：返回一个新的 `promise`，只有所有的 `promise` 都成功才成功，只要有一个失败了就直接失败

```javascript
let p1 = new Promise((resolve, reject) => {
  resolve('OK');
})
let p2 = Promise.resolve('Success');
let p3 = Promise.resolve('Oh Yeah');

const result = Promise.all([p1, p2, p3]);
console.log(result);
```

<img src="https://img-blog.csdnimg.cn/2021022621374890.png" alt="2021022621374890" style="zoom:90%;" align="left" />

##### 7.Promise.race方法：`Promise.race(iterable)`

iterable：包含 n 个 `promise`的可迭代对象，如`Array`或`String`

说明：返回一个新的`promise`，第一个完成 `promise`的结果状态就是最终的结果状态

**谁先完成(改变状态)就输出谁(不管是成功还是失败)**

```javascript
const pRace = Promise.race([p1, p2, p3])
// 谁先完成就输出谁(不管是成功还是失败)
const p1 = new Promise((resolve, reject) => {
 setTimeout(() => {
   resolve(1)
 }, 1000)
})
const p2 = Promise.resolve(2)
const p3 = Promise.reject(3)

pRace.then(
value => {
   console.log('race onResolved()', value)
 },
reason => {
   console.log('race onRejected()', reason) 
 }
)
//race onResolved() 2
```



### 三.Promise的几个关键问题

#### 1.如何改变promise状态

(1)`resolve(value)`：如果当前是 `pending` 就会变为 `resolved`

(2)`reject(reason)`：如果当前是 `pending` 就会变为 `rejected`

(3)抛出异常：如果当前是 `pending` 就会变为 `rejected`

#### 2. 一个 promise 指定多个成功/失败回调函数，都会调用吗？

会的

```javascript
const p = new Promise((resolve, reject) => {
  //resolve(1)
  reject(2)
})
p.then(
  value => {},
  reason => {console.log('reason',reason)}
)
p.then(
  value => {},
  reason => {console.log('reason2',reason)}
)
// reason 2
// reason2 2
```

#### 3. 改变promise状态和指定回调函数谁先谁后？

> 都有可能，常规是先指定回调再改变状态，但也可以先改状态再指定回调

先指定回调再改变状态时，then虽然会执行，但不会执行成功或者失败的函数，then会等待，因为Promise此时的状态还是pending

- 如何先改状态再指定回调？

(1)在执行器中直接调用 `resolve()`/`reject()`

(2)延迟更长时间才调用 `then()`

```javascript
let p = new Promise((resolve, reject) => {
  // setTimeout(() => {
      resolve('OK');
  // }, 1000); // 有异步就先指定回调，否则先改变状态
});

p.then(value => {
  console.log(value);
},reason=>{
})
```



 #### 4.promise.then() 返回的新 promise 的结果状态由什么决定？

(1)简单表达：由 `then()` 指定的回调函数执行的结果决定

(2)详细表达：

 ① 如果抛出异常，新`promise`变为`rejected`，`reason`为抛出的异常

 ② 如果返回的是非`promise`的任意值， 此`promise`变为`resolved`，`value`为返回的值

 ③ 如果返回的是另一个新 `promise`，此`promise`的结果就会成为新`promise`的结果



#### 5.promise 如何串联多个操作任务？

(1)`promise`的`then()`返回一个新的`promise`，可以并成`then()`的链式调用

(2)通过 `then` 的链式调用串联多个同步/异步任务

```javascript
let p = new Promise((resolve, reject) => {
  setTimeout(() => {
      resolve('OK');
  }, 1000);
});

p.then(value => {
  return new Promise((resolve, reject) => {
      resolve("success");
  });
}).then(value => {
  console.log(value); // success
}).then(value => {
  console.log(value); // undefined
})
```

#### 6.Promise 异常穿透(传透)？

(1)当使用 `promise` 的 `then` 链式调用时，可以在最后指定失败的回调

(2)前面任何操作出了异常，都会传到最后失败的回调中处理

```javascript
new Promise((resolve, reject) => {
   //resolve(1)
   reject(1)
}).then(
  value => {
    console.log('onResolved1()', value)
    return 2
  }
).then(
  value => {
    console.log('onResolved2()', value)
    return 3
  }
).then(
  value => {
    console.log('onResolved3()', value)
  }
).catch(
  reason => {
    console.log('onRejected1()', reason)
  }
)
// onRejected1() 1
```

 #### 7.中断 promise 链

当使用`promise`的`then`链式调用时，在中间中断，不再调用后面的回调函数

办法：在回调函数中返回一个`pending`状态的`promise`对象,因为状态没改变，所以后面的then里的内容和catch就不会执行

```javascript
let p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("OK");
  }, 1000);
});

p.then((value) => {
  console.log(111);
  //有且只有一个方式
  return new Promise(() => {});
})
  .then((value) => {
    console.log(222);
  })
  .then((value) => {
    console.log(333);
  })
  .catch((reason) => {
    console.warn(reason);
  });
//111
```



### 四.手写Promise(暂时不学)



### 五.async和await

#### 1.async和await

`async`和`await`是一种语法糖，两种语法结合可以让异步代码像同步代码一样

`async`和`await`关键字让我们可以用一种更简洁的方式写出基于`Promise`的异步行为，而无需刻意地链式调用`promise`

`async`用来声明异步函数，函数会返回一个`promise`对象

`await`用来暂停异步函数代码的执行，等待`promise`解决



#### 2.async声明异步函数

1.`async`函数返回为`Promise`对象

2.`promise`对象的结果由`async`函数执行的返回值决定

```javascript
async function fn() {
 // 1. 返回结果不是一个Promise类型的对象，返回的结果就是成功的Promise对象
 // return 'yk';  // Promise {<resolved>: "yk"}
  // return; // Promise {<resolved>: undefined}
	
  // 2. 抛出错误，返回的结果是一个失败的Promise
  // throw new Error('出错'); // Promise {<rejected>: Error: 出错

  // 3. 返回一个Promise
  return new Promise((resolve, reject) => {
    // resolve('成功的数据');
    reject('失败的数据');
  })
}

const result = fn();
console.log(result);

result.then(value => {
  console.log(value);
}, reason => {
  console.warn(reason);
})
```

#### 3.await表达式

> 异步函数主要针对不会马上完成的任务，所以自然需要一种暂停和恢复执行的能力，使用`await`关键字可以暂停异步函数代码的执行，等待期约解决

1. `await` 必须写在`async`函数中，但是`async`函数中可以没有`await`
2. `await` 右侧的表达式一般为 `promise` 对象，也可以是其他值
3. `await` 返回的是`promise`成功的值，如果是其他值就返回此值
4. `await`的`promise`失败了，就会抛出异常，需要通过`try-catch`捕获处理

```javascript
const promise = new Promise((resolve, reject)=> {
  // resolve('成功的值');
  reject('失败的数据');
})

async function main(){
  try {
    let result = await promise;
    console.log(result); // 成功的值
  } catch(e){
    console.log(e); // 失败的数据
  }
}
main()  
```

