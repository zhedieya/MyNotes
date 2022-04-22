[toc]

---

#### import错误

在跟着尚硅谷学vue时，点击'随机一下'时报错，逻辑是从用axios从接口获取数据添加到列表里



根据报错提示，可以知道错误是发生在nanoid上的

随后检查了一下代码里关于nanoid的地方，发现**import调用格式错误**

```js
import nanoid from 'nanoid'
```

应该写成：

```js
import {nanoid} from 'nanoid'
```

**加上花括号！！！**

##### 补充知识点：import的导入规范

1. 当你默认导出一个变量或方法等的时候，也就是你在模块A中`export default a`，那么你导入时，始终不需要花括号，即：

   ```js
   //模块A中
   const a = 12
   export default a
    
   //模块B中
   import a from './A'  //这边的 a 可以重命名，不影响使用，所以可以这么写
   import every from './A'
   ```

2. 但是如果你没有默认导出，而是通过`export const a`这种方式的话，导出是需要拿到模块中具体变量的，因此你需要通过解构这种方式来拿到模块中具体的变量。即：

   ```js
   //模块A中
   export const a = 12
    
   //模块B中
   import {a} from './A'  //这时的a是解构出的具体变量，因此不可以直接起别名，通过下面这种方式重命名
   import {a as another}from './A'  //这个时候在模块B中可以用another来使用A中定义的模块和方法
   ```

3. 简单总结，默认导出，不需要花括号。

---

