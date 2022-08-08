[toc]

---

### 插槽slot

[插槽](https://staging-cn.vuejs.org/guide/components/slots.html)

1. 作用：让父组件可以向子组件指定位置插入html结构，也是一种组件间通信的方式，适用于 `父组件 ===> 子组件`

2. 分类：默认插槽、具名插槽、作用域插槽

3. 使用方式：

#### 1.默认插槽

父组件中

```html
<Category>
   <div>html结构1</div>
</Category>
```

子组件中

```html
<template>
    <div>
       <!-- 定义插槽 -->
       <slot>插槽默认内容...</slot>
    </div>
</template>
```

#### 2.具名插槽

> '具有名字的插槽'

要为具名插槽传入内容，我们需要使用一个含 `v-slot` 指令的 `<template>` 元素，并将目标插槽的名字传给该指令

父组件中

```vue
<Category>
    <template #center>
      <div>html结构1</div>
    </template>

    <template v-slot:footer>
       <div>html结构2</div>
    </template>
</Category>
```

子组件中

```html
<template>
    <div>
       <!-- 定义插槽 -->
       <slot name="center">插槽默认内容...</slot>
       <slot name="footer">插槽默认内容...</slot>
    </div>
</template>
```



#### 3.作用域插槽

1. 理解：**数据在组件的自身，但根据数据生成的结构需要组件的使用者来决定。**（games数据在Category组件中，但使用数据所遍历出来的结构由App组件决定）

2. 具体编码：

   > 父组件中

   ```vue
   <Category>
       <template v-slot="scopeData"> 
       <!-- 或者以解构的形式获取数据 -->
       <template v-slot="{games}">
           <!-- 生成的是ul列表 -->
           <ul>
               <li v-for="g in scopeData.games" :key="g">{{g}}</li>
           </ul>
       </template>
   </Category>
   
   <Category>
        <!-- Vue2 -->
       <template slot-scope="scopeData">
           <!-- 生成的是h4标题 -->
           <h4 v-for="g in scopeData.games" :key="g">{{g}}</h4>
       </template>
   </Category>
   
   ```

3. > 子组件中    数据在子组件自身

   ```vue
   <template>
       <div>
          <!-- 将数据回传给父组件 -->
           <slot :games="games"></slot>
       </div>
   </template>
   
   <script>
       export default {
           name:'Category',
           props:['title'],
           //数据在子组件自身
           data() {
               return {
                   games:['红色警戒','穿越火线','劲舞团','超级玛丽']
               }
           },
       }
   </script>
   
   ```

> 可以想象的是，一些组件可能只包括了逻辑而不需要自己渲染内容，视图输出通过作用域插槽全权交给了消费者组件。我们将这种类型的组件称为**无渲染组件**。

##### [具名作用域插槽](https://staging-cn.vuejs.org/guide/components/slots.html#named-scoped-slots)

具名作用域插槽的工作方式也是类似的，插槽 props 可以作为 `v-slot` 指令的值被访问到：`v-slot:name="slotProps"`。当使用缩写时是这样：

```vue
<MyComponent>
  <template #header="headerProps">
    {{ headerProps }}
  </template>

  <template #default="defaultProps">
    {{ defaultProps }}
  </template>

  <template #footer="footerProps">
    {{ footerProps }}
  </template>
</MyComponent>
```

向具名插槽中传入 props：

```
<slot name="header" message="hello"></slot>
```

注意插槽上的 `name` 是一个 Vue 特别保留的 attribute，不会作为 props 传递给插槽。因此最终 `headerProps` 的结果是 `{ message: 'hello' }`。

#### 总结

**插槽的默认内容：**如果希望在使用插槽时，如果没有插入对应的内容，需要显示一个默认的内容。这个默认的内容只会在没有提供插入的内容时，才会显示。

**在不使用具名插槽的情况下：**如果一个组件中含有多个插槽，在使用组件时插入多个内容时是什么效果？会发现默认情况下每个插槽都会获取到我们插入的内容来显示（即：每个插槽中都会放入插入的多个内容）。所以才需要具名插槽。

具名插槽顾名思义就是给插槽起一个名字，在使用时可以指定内容插入到指定的插槽中。\<slot> 元素有一个特殊的 attribute：**name**。<font color=FF0000>一个不带 name 的 slot，会带有隐含的名字 default</font>。另外，具名插槽v-slot 有缩写 #

**动态插槽名**：目前我们使用的插槽名称都是固定的，比如 v-slot:left、v-slot:center等。可以通过 **v-slot:[dynamicSlotName]** 方式动态绑定一个名称（<font size=4>**这里类似于JS中的计算属性**</font>）。**使用场景：**一般用于高级组件，<font color=FF0000>通过配置绑定插槽名</font>，比如父组件使用 prop 将插槽名称传递给子组件；子组件在写的时候自动绑定。

**渲染作用域：**在Vue中有渲染作用域的概念，<font color=FF0000>父级模板里的所有内容都是在父级作用域中编译的，子模板里的所有内容都是在子作用域中编译的</font>。<font size=4>**有时也会希望插槽可以访问到子组件中的内容：** </font>当一个组件被用来渲染一个数组元素时，我们使用插槽，并且希望插槽中没有显示每项的内容；<font color=FF0000 size=4>**Vue给我们提供了作用域插槽（插槽prop）**</font>

代码示例如下：

[Vue SFC](https://sfc.vuejs.org/#eNqNVN9v0zAQ/leOTKib1CTdxmCErhIPwMvEyyYktOzBTS6pN8e2bKeslP7vnJM6bccESFGUO/v77td3WUcftU6WLUZZNLWF4drNcskbrYyDz0wWq2tuHVRGNTBK0sHjIaNcAuQSn7rLJVasFQ7W3lsoYpAonc16B+zIvLnJJT3TdIhIhsNGC+aQLIDpLjbTPG6NuMojeucRZBpNrFmN5Dmd5FF3nxABD0ecPulwDXNVrsbQWjSSNTgGwR/RUvCAIVTJl1AIZi0BPG7vjE71bN2zwGYzTfXB0QBr0DGCzVdAl0MwAsAv79jG3PQf+yTTlIKH7NPD8nedJvvgkEzrVgLBFkpjSZ7EJ9C3uVLSxZb/xAwmySU2H/phCGUyOHpzNr+8PCdX33pPMovGUT/suGE6ebBKkhA6KmpHd2DzaJhhHtHYvZ1HC+e0zdLUVoXXwoNNlKlT+kpMKx1vMEHbxHOjflBDiDiPxnscKTmXNEaDskSD5m+cz67+wRv0RKUcyPNA0S+JVBtF4eButFXYaAyjoK3RfZduyRw7PgnlG3StkcEC8HrxBPe9g5KgV4drFDUByx00TclXPBJFo0iinhcqdMWCy7q/YdHdUttU644JdjXbhXELbpMuFlzB3U6CvTIzGN10SoAboZyFLy0vkUoJQqTzT0sm4btqyduJMIOzyTbRZ0zfWoTb1inDmW/HHsdXylhwBreoRUtZsx3ZKZEFrtCKMXknk5P/2/VWhC0QHJYxr2ipXnUFJwJl7RZ7S3mtWEk9S5IkLI7gB+hKme0qA5f9iPb33VKTwNc07Dth5lyWg50OK7llnqZdgv/YQoBWhJkJkmDcncdupal9kn6G3S6S6ljpC8jgQj9tXXNWPNaGJFNmBJXITFwbKpP+n8fnpxcl1mO/veX5+zM4u3hNxts377CqTjp813sqfRt7n5/mPARpmKm59NN6Me5RVVWBbvg5bH4DB0X6SA==) (更推荐看这个)

```html
<!-- showNames.vue -->

<!-- 这里的name是app.vue传来的，还要通过下面的slotProp传回去；当然，slotProps的名字是可以改的 -->
<template v-for="(item, index) in names" :key="item">
	<slot :item="item" :index="index"> </slot>
</template>

<script>
	export default {
    props: {
      data: {
        type: Array,
        default: () => []
      }
    }
  }
</script>
```

```html
<!-- app.vue -->
<show-name :names="names">
  <!-- 接收showName.vue传来的 slotProps，并使用 -->
  <!-- 如果插槽有名字，比如<slot name="foo" ...></slot>，这时候，下面的v-slot需要加上名字；即：v-slot:foo="slotProps"。另外，由于插槽没有名字，也叫default、并被省略；即也可以被写作：v-slot:default="slotProps" --> 
	<template v-slot="slotProps">
  	<button>{{slotProps.item}}-{{slotProps.index}}</button>
  </template>
</show-name>

<script>
	export default {
    data() {
      return {
        names: ['foo', 'bar', 'baz', 'quz']
      }
    }
  }
</script>
```

**独占默认插槽的缩写**

由于上面需要加上 \<template v-slot="slotProps">比较麻烦，所以可以去掉template；不过这是有条件的

<font color=FF0000 size=4>**当且仅当，这个插槽是一个独占默认插槽时才可使用，即这个插槽是组件中唯一的插槽。如果还有其他具名插槽，并且使用了这个插槽，则不可以这样写。还是要分别用\<tempalte>封装，并使用插槽**</font>

```html
<!-- app.vue -->
<show-name :names="names" v-slot="slotProps">
	<button>{{slotProps.item}}-{{slotProps.index}}</button>
</show-name>
```



### 