[toc]

### WeakMap

#### WeakMap 的特性

##### 1. WeakMap 只接受对象作为键名

```js
const map = new WeakMap();
map.set(1, 2); // TypeError: Invalid value used as weak map key
map.set(null, 2); // TypeError: Invalid value used as weak map key
```

##### 2. WeakMap 的键名所引用的对象是弱引用

这句话其实让我非常费解，我个人觉得这句话真正想表达的意思应该是：

> WeakMaps hold "weak" references to key objects

翻译过来应该是 <font color=FF0000>WeakMaps 保持了 **对键名所引用的对象的弱引用**</font>。

我们先聊聊弱引用：

> <mark>在计算机程序设计中，**弱引用与强引用相对**</mark>，是指 <font color=FF0000>**不能确保** 其引用的对象 **不会被垃圾回收器回收** 的引用</font>。 <font color=FF0000>一个对象若只被弱引用所引用，**则被认为是 不可访问**（或弱可访问）的，并 **因此可能在任何时刻被回收**</font>。

在 JavaScript 中，<font color=FF0000>一般我们创建一个对象，**都是建立一个强引用**</font>：

```js
var obj = new Object();
```

<font color=FF0000>只有当我们手动设置 `obj = null` 的时候，**才有可能回收 obj 所引用的对象**</font>。

而如果我们能创建一个弱引用的对象：

```js
// 假设可以这样创建一个
var obj = new WeakObject();
```

我们<font color=FF0000>什么都不用做，只用静静的等待垃圾回收机制执行，obj 所引用的对象就会被回收</font>。

我们再来看看这句：

> WeakMaps 保持了对键名所引用的对象的弱引用

正常情况下，我们举个例子：

```js
const key = new Array(5 * 1024 * 1024);
const arr = [ [key, 1] ];
```

使用这种方式，我们<mark>其实建立了 arr 对 key 所引用的对象</mark>（我们假设这个真正的对象叫 Obj ）<mark>的 **强引用**</mark>。

所以<font color=FF0000>**当你设置 `key = null` 时，只是去掉了 key 对 Obj 的强引用**</font>，<font color=FF0000>并 **没有去除 arr 对 Obj 的强引用，所以 Obj 还是不会被回收掉**</font>。

**Map 类型也是类似：**

```js
let map = new Map();
let key = new Array(5 * 1024 * 1024);

map.set(key, 1); // 建立了 map 对 key 所引用对象的强引用
key = null;      // key = null 不会导致 key 的原引用对象被回收
```

我们依然通过 Node 证明一下：

```js
// 注：假设文件名为 node-gc.js
global.gc();
process.memoryUsage(); // heapUsed: 4638376 ≈ 4.4M

let map = new Map();
let key = new Array(5 * 1024 * 1024);
map.set(key, 1);
global.gc();
process.memoryUsage(); // heapUsed: 46727816 ≈ 44.6M

map.delete(key);
global.gc();
process.memoryUsage(); // heapUsed: 46748352 ≈ 44.6M

key = null;
global.gc();
process.memoryUsage(); // heapUsed: 4808064 ≈ 4.6M
```

```sh
node --expose-gc node-gc.js # 注：注意 --expose-gc 必需放在 js 文件名前
```

**注：**要查看运行结果可以在 `process.memoryUsage();` 外面包裹一层 `console.log()`。打印结果为：

```js
{ rss: 27213824, heapTotal: 6070272, heapUsed: 3503776, external: 447189, arrayBuffers: 10430 }
{ rss: 69857280, heapTotal: 48553984, heapUsed: 45782136, external: 457993, arrayBuffers: 10430 }
{ rss: 69955584, heapTotal: 50651136, heapUsed: 45788184, external: 457993, arrayBuffers: 10430 }
{ rss: 70057984, heapTotal: 50651136, heapUsed: 45788296, external: 457993, arrayBuffers: 10430 }
```

如果你想要让 Obj 被回收掉，你需要先 `delete(key)` 然后再 `key = null`:

```js
let map = new Map();
let key = new Array(5 * 1024 * 1024);
map.set(key, 1);
map.delete(key);
key = null;
```

我们依然通过 Node 证明一下（ 👀 **注：**代码略，见原文）

这个时候就要说到 WeakMap 了：

```js
const wm = new WeakMap();
let key = new Array(5 * 1024 * 1024);
wm.set(key, 1);
key = null;
```

当我们 <font color=FF0000>设置 `wm.set(key, 1)` 时，其实建立了 wm 对 key 所引用的对象的弱引用</font>，<mark>但因为 `let key = new Array(5 * 1024 * 1024)` **建立了 key 对所引用对象的强引用**，被引用的对象并不会被回收</mark>，但是<font color=FF0000>**当我们设置 `key = null` 的时候，就只有 wm 对所引用对象的弱引用**，下次垃圾回收机制执行的时候，该引用对象就会被回收掉</font>。

我们用 Node 证明一下：

```js
global.gc();
process.memoryUsage(); // heapUsed: 4638992 ≈ 4.4M

const wm = new WeakMap();
let key = new Array(5 * 1024 * 1024);
wm.set(key, 1);
global.gc();
process.memoryUsage(); // heapUsed: 46776176 ≈ 44.6M

key = null;
global.gc();
process.memoryUsage(); // heapUsed: 4800792 ≈ 4.6M
```

所以 <font color=FF0000>WeakMap 可以帮你省掉手动删除对象关联数据的步骤</font>，所以当你不能或不想控制关联数据的生命周期时就可以考虑使用 WeakMap

总结这个弱引用的特性，就是 WeakMaps 保持了对键名所引用的对象的弱引用，即垃圾回收机制不将该引用考虑在内。<font color=FF0000>只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存</font>。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。

也正是因为这样的特性，WeakMap 内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一样的，而 <font color=FF0000 size=4>**垃圾回收机制何时运行是不可预测的**，**因此 ES6 规定 WeakMap 不可遍历**</font>。

所以 <font color=FF0000>WeakMap 不像 Map，一是没有遍历操作（即没有 keys()、values() 和 entries() 方法），也没有 size 属性，也不支持 clear 方法</font>；所以 <font color=FF0000>**WeakMap 只有四个方法可用**：get()、set()、has()、delete()</font>。

#### 应用

##### 1. 在 DOM 对象上保存相关数据

传统使用 jQuery 的时候，我们会<font color=FF0000>通过 \$.data() 方法在 DOM 对象上储存相关信息</font>（就比如在删除按钮元素上储存帖子的 ID 信息），jQuery 内部会使用一个对象管理 DOM 和对应的数据，当你将 DOM 元素删除，DOM 对象置为空的时候，相关联的数据并不会被删除，你必须手动执行 \$.removeData() 方法才能删除掉相关联的数据。<font color=FF0000>**WeakMap 就可以简化这一操作**</font>：

```js
let wm = new WeakMap(), element = document.querySelector(".element");
wm.set(element, "data");

let value = wm.get(elemet);
console.log(value); // data

element.parentNode.removeChild(element);
element = null;
```

##### 2. 数据缓存

从上一个例子，我们也可以看出，当我们需要关联对象和数据，比如在不修改原有对象的情况下储存某些属性或者根据对象储存一些计算的值等，而又不想管理这些数据的死活时非常适合考虑使用 WeakMap。数据缓存就是一个非常好的例子：

```js
const cache = new WeakMap();
function countOwnKeys(obj) {
    if (cache.has(obj)) {
        console.log('Cached');
        return cache.get(obj);
    } else {
        console.log('Computed');
        const count = Object.keys(obj).length;
        cache.set(obj, count);
        return count;
    }
}
```

##### 3. 私有属性

<font color=FF0000>WeakMap 也可以被用于实现私有变量</font>，不过在 ES6 中实现私有变量的方式有很多种，这只是其中一种：

```js
const privateData = new WeakMap();

class Person {
    constructor(name, age) { 
      privateData.set(this, { name: name, age: age }); 
    }
    getName() { return privateData.get(this).name; }
    getAge() { return privateData.get(this).age; }
}

export default Person;
```

摘自：[ES6 系列之 WeakMap](https://github.com/mqyqingfeng/Blog/issues/92)
