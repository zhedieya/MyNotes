大家好哇，这里是前端新手zhedieya(折叠鸭)，在深思熟虑后呢，我决定开始在掘金输出知识来巩固和总结自己遇到的技术和问题

今天给大家带来的是个非常简单的问题----el-table复选框与行选中的联调问题，用大白话来说，就是勾选复选框时高亮行，行选中时勾选复选框，以及该问题所延伸出来的一些小问题！

>Don’t ignore your dreams; don’t work too much; say what you think; cultivate friendships; be happy.

### 预备知识

| 事件名/方法名/参数 | 说明                                                         | 参数                             |
| ------------------ | ------------------------------------------------------------ | -------------------------------- |
| @selection-change  | 当选择项发生变化时会触发该事件                               | selection                        |
| @row-click         | 当某一行被点击时会触发该事件                                 | row, column, event               |
| toggleRowSelection | 用于多选表格，切换某一行的选中状态，如果使用了第二个参数，则是设置这一行选中与否（selected 为 true 则选中） | row, selected                    |
| setCurrentRow      | 用于单选表格，设定某一行为选中行，如果调用时不加参数，则会取消目前高亮行的选中状态。 | row                              |
| row-style          | 行的 style 的回调方法，也可以使用一个固定的 Object 为所有行设置一样的 Style。 | Function({row, rowIndex})/Object |



### 问题背景

最近本人在写公司项目时，遇到了个看上去很简单的项目bug，在业务将bug从禅道分配给我后，我瞅了眼，是关于el-table的问题：复选框选中表格行数据进行进一步处理时(这儿以一键三连为例)，报错提示显示没有选中到数据；

在我定位到代码后，仔细排查了下问题，发现是操作逻辑的问题，触发一键三连事件的数据，是通过「row-click」也就是行选中来触发的，而复选框是通过「selection-change」



<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20220728225118053.png" alt="image-20220728225118053" style="zoom:50%;" />

如图所示，一键三连事件是通过选中行「row-click」来触发的，是单选；而zhedieya事件是通过勾选复选框触发「selection-change」来触发的，是多选；至于在最初设计的时候

为什么要区分开事件，可能就是因为这个单选多选的原因吧。

在把问题告诉项目组长后，项目组长示意我可以将功能合并，在行选中的时候也触发该行的复选框，反之，勾选复选框时也触发该行的行选中,来统一获取数据处理事件。

于是，我就开始了头脑风暴(bushi)

### 解决问题

##### 最初的想法

这是最初的el-table属性

```vue
<el-table
          ref="multipleTable"
          :data="tableData"
          border
          style="width: 100%"
          @selection-change="dataListSelectionChangeHandle"
          @row-click="handleRowClick"
          highlight-current-row
        >
</el-table>
```

- 1.触发行选中时，同时选中复选框

  通常行选中事件row-click往往伴随着highlight-current-row属性使用，增加了highlight-current-row属性，在行选中时，被选中的行会有个默认的颜色变化。

  只需在row-click事件中添加如下代码便可

  ```js
  handleRowClick(row) {
     this.$refs.multipleTable.toggleRowSelection(row)；
  },
  ```

  <img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20220728230451388.png" alt="image-20220728230451388" style="zoom:50%;" />

  如图所示，达到了这一效果

  **问题：**选中后再次点击时，只会变更复选框状态，而行依旧是选中后的高亮状态

  这是小事儿，因为可以通过一些手段判断是否为第二次点击，可通过

  ```js
  this.$refs.multipleTable.setCurrentRow();
  ```

  来取消目前高亮行的选中状态

  但最主要的问题还是：

- 2.选中复选框时，触发行选中

  奈何我水平有限，没找到勾选复选框所能牵扯到的行选中的属性，所以这个想法就不了了之了(这时候还是死脑筋，非得揪着以行选中的「属性」来变更)。

  

##### 解决的方法

后来出去上看来个厕所透了气，灵感就这么出来了，是我之前想多啦，可以①直接在「勾选复选框」的同时更改那一行的颜色，同时，还能②在「行选中」的同时「勾选复选框」，这就又

回到了①更改掉了行的颜色，实现了我们想要的效果。

el-table属性：

```vue
<el-table
          ref="multipleTable"
          :data="tableData"
          border
          style="width: 100%"
          @selection-change="dataListSelectionChangeHandle"
          @row-click="handleRowClick"
          :row-style="rowClass"
          >
</el-table>
```

记得这里要去掉highlight-current-row属性

「行选中」触发的事件，`@row-click`：

```js
// 选中行
handleRowClick(row) {
  // 切换这一行复选框的选中状态
  this.$refs.multipleTable.toggleRowSelection(row)
},
```

「勾选复选框」触发的事件，`@selection-change`：

```js
// 选中复选框
dataListSelectionChangeHandle(val) {
  this.multipleSelection = val; // 处理数据
},
```

> mutipleSelection是数组对象，值为当前勾选住的行的信息

ok，基础完成了，接下来就是样式的变化，在这里我们可以使用el-table内置属性`@row-style`，这是行的 style 的回调方法，可以使用一个固定的 Object 为所有行设置一样的 Style；

这里使用`this.multipleSelection.includes(row)`来判断出被选中的行，进行样式的更改

```js
rowClass({ row, rowIndex }) {
  // 更换被选中行的背景颜色，且不指定--tablebackground,这样hover取消掉了hover的默认颜色
  if (this.multipleSelection.includes(row)) {
    return {
      "background-color": "#73caa3",
    };
  } else {
    // 没被选中的行hover颜色是初始颜色
    return {
      "--tablebackground": "#F5F7FA",
    };
  }
},
```

不过这还没完，因为el-table-column有个默认的hover样式，所以行选中后的行的背景颜色会被hover后的背景颜色覆盖，只有鼠标移开，被选中的行才会呈现出该有的颜色，所

以我们要「清除或者动态更改」hover的样式

清除的话非常简单，直接将hover样式后的背景颜色改为`transparent`便可(不过存在个弊端：行没有被选中时hover样式消失了)

我这儿是想保留默认hover的样式，所以就通过自定义属性来获取值，动态修改hover后的颜色

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20220729230119211.png" alt="image-20220729230119211" style="zoom:50%;" />

```css
//没有被固定住列的表格
::v-deep .el-table tbody tr:hover > td {
  background-color: var(--tablebackground) !important;
} 
//有被固定住列的表格
::v-deep .el-table .el-table__body tr.hover-row > td {
  background-color: var(--tablebackground) !important;
}
```

> 这儿也有个要注意的点，若某Table-column加了属性fixed而被固定，需要使用上面第二个css选择方式，否则会出现fixed的行hover样式依旧存在的情况

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20220729225632792.png" alt="image-20220729225632792" style="zoom:50%;" />

### 总结

一句话可以概况：功力尚浅，仍需学习

最后，放个我最近在磕的双周CP来收个尾吧！！！！

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/IMG_0853.JPG" alt="IMG_0853" style="width:50%;" align="left" />