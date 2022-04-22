[toc]

##### HTML语义化

>1、易于用户阅读，css样式丢失的时候能让页面呈现清晰的结构。
>
>2、有利于SEO，搜索引擎根据标签来确定上下文和各个关键字的权重。
>
>3、方便其他设备解析，如盲人阅读器根据语义渲染网页
>
>4、有利于开发和维护，语义化更具可读性，代码更好维护，与CSS3关系更和谐  



##### 页面渲染流程

>1．解析HTML文件，创建DOM树
>
>2．解析CSS，形成CSS对象模型
>
>3．将CSS与DOM合并，构建渲染树（renderingtree）
>
>4．回流与重绘  （重排 重绘)
>
>5. Display 


<img src="/Users/owsl/Library/Application Support/typora-user-images/image-20220323101801278.png" alt="image-20220323101801278" style="zoom:40%;" align="left"/>



##### 回流(重排)与重绘

1. 渲染过程：DOM树（HTML）& CSSOM （CSS） 结合生成渲染树；回流（Layout）得到节点的几何信息（大小和位置）；重绘（Painting）得到节点的绝对像素，将渲染树的每个节点都转换为屏幕上的实际像素；展示Display。

2. 渲染树只包含可见节点， 一些不会渲染输出的节点，比如 script、meta、link 等 和 设置 display:none 的节点不会出现在渲染树中。但是利用 visibility 和 opacity 隐藏的节点会显示在渲染树上。

3. 当页面布局和几何信息发生变化的时候，就需要回流。如新增 DOM，元素尺寸变化，窗口大小变化等。回流一定会触发重绘，而重绘不一定会回流。

4. 大多数浏览器都会通过队列化修改并批量执行来优化重排过程。当你获取布局信息的操作的时候，会强制队列刷新。

5. 减少回流和重绘：

   1）修改样式的时候通过 css 类名修改或通过 cssText 修改。

   2）DOM 元素离线修改—>隐藏元素，应用修改，重新显示。（浏览器本身也会有优化）

   3）避免触发同步布局事件，如获取 offsetWidth 等属性值，因为会强制浏览器刷新队列。

   4）使用绝对定位让复杂动画脱离文档流减少父元素以及后续元素频繁的回流。

   5）使用css3硬件加速，可以让 transform、opacity、filters、will-change 这些动画不会引起回流重绘 （会提高内存占用）。

##### 为什么CSS要放在头部，js要放在body底部

>因为css是要在网页渲染外观的时候就要调用的，所以要预先调入内存，因此放在头部。
>
>而js因为有可能需要调用网页DOM结构中的元素，所以必须等HTML的整个DOM结构都调入内存后才开始运行（否则就很可能因为无法找到元素而出错），所以要放在底部。
>
>但是最好或者说最标准的用法是把js代码放入window对象的onload事件中，这样就可以把整个js代码放到网页的任何位置了（包括头部）



##### 清除浮动

```css
1.开启BFC
2.clear:both清除浮动元素对当前元素所产生的影响
3.使用after before伪元素清除浮动
```



##### BFC块级格式化上下文

>BFC 即块级格式上下文，根据盒模型可知，每个元素都被定义为一个矩形盒子，然而盒子的布局会受到**尺寸，定位，盒子的子元素或兄弟元素，视口的尺寸**等因素决定
>
>它是页面中的一块渲染区域，具有BFC特性的元素可以看作隔离了的容器，容器里面的元素在布局上不会影响到外面的。
>
>产生条件：
>
>- 根元素（`<html>`）
>- 浮动元素（`float` 不为 `none`）
>- 绝对定位元素（`position` 为 `absolute` 或 `fixed`）
>- 表格的标题和单元格（`display` 为 `table-caption`，`table-cell`）
>- 匿名表格单元格元素（`display` 为 `table` 或 `inline-table`）
>- 行内块元素（`display` 为 `inline-block`）
>- `overflow` 的值不为 `visible` 的元素
>- 弹性元素（`display` 为 `flex` 或 `inline-flex` 的元素的直接子元素）
>- 网格元素（`display` 为 `grid` 或 `inline-grid` 的元素的直接子元素）
>
>以上是 `CSS2.1` 规范定义的 `BFC` 触发方式，在最新的 `CSS3` 规范中，弹性元素和网格元素会创建 `F(Flex)FC` 和 `G(Grid)FC`。
>
>链接：https://juejin.cn/post/6960866014384881671
>
>
>特点：BFC可以包含浮动的元素，清除浮动 (从而可以避免高度塌陷)   
>
>​     BFC 可以阻止元素被浮动元素覆盖  
>
>​     不同的BFC可以防止外边距重叠



##### Flex

 [Flex 布局教程](https://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

- `flex-direction` ：决定主轴的方向
- `flex-wrap` ：如果一条轴线排不下，如何换行
- `flex-flow` ：`flex-direction`属性和`flex-wrap`属性的简写形式，默认值为`row nowrap`
- `justify-content` ：元素在主轴的对齐方式
- `align-items` ：元素在副轴的对齐方式
- `align-content` ：定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用

```css
flex:     grow shrink basis;
 initial "flex: 0 1 auto".
 auto    "flex: 1 1 auto"
 none    "flex: 0 0 auto" 
 1       "flex: 1 1 0% " 
```

- `flex-grow: 1` ：该属性默认为 `0` ，如果存在剩余空间，元素也不放大。设置为 `1` 代表会放大。
- `flex-shrink: 1` ：该属性默认为 `1` ，如果空间不足，元素缩小。
-  flex-basis: 0%` ：该属性定义在分配多余空间之前，元素占据的主轴空间。浏览器就是根据这个属性来**计算是否有多余空间**的。默认值为 `auto` ，即项目本身大小。设置为 `0%` 之后，因为有 `flex-grow` 和 `flex-shrink` 的设置会自动放大或缩小。在做两栏布局时，如果右边的自适应元素 `flex-basis` 设为 `auto` 的话，其本身大小将会是 `0` 。



##### 一个盒子垂直水平居中有哪些方法？

1.绝对定位后transform

```css
.parent{
  position:relative;
}
.child{
  position:absolute;
  top:50%;
  left:50%;
  transform:translate(-50%,-50%);
}
```

2.flex

```css
{
  display:flex;
  justify-content:center;
  align-items:center;
}
```

3.margin:auto（必须有宽高）

```css
.parent{ 
  position:relative;
}

.child{
  position:absolute;
  margin:auto;
  top:0;
  left:0;
  right:0;
  bottom:0;
  height: 100px;
  width: 100px;
}
```



##### 自适应布局

- **两栏布局左边固定，右侧自适应，两列等高**

  <font color=darkblue>1.将左侧div浮动，右侧div设置margin-left</font>

  ```css
  .outer {
     overflow: hidden;
     border: 1px solid red;
  }
  
  .sidebar {
     float: left;
     width: 200px;
     height: 150px;
     background: #BCE8F1;
  }
  
  .content {
     margin-left: 200px;
     height: 150px;
     background: #F0AD4E;
  }
  ```

  <font color=darkblue>2.float+BFC</font>

  ```css
   .outer{
      border: 1px solid red;
  }
  
   .sidebar {
      float: left;
      height: 150px;
      background: #BCE8F1;
  }
  
    .content {
      overflow: auto;
      height: 100px;
      background: #F0AD4E;
  }
  ```
  
  <font color=darkblue>3.flex </font> 左边元素固定宽度，右边的元素设置 `flex: 1`
  
  ```css
  .outer {
     display: flex;
     border: 1px solid red;
  }
  
  .sidebar {
    flex: 0 0 200px;
    height: 150px;
    background: #BCE8F1;
  }
  
  .content {
    flex: 1;
    height: 100px;
    background: #F0AD4E;
  }
  ```
  
  
  
- **三栏布局多种写法:圣杯、双飞翼**

  [圣杯布局中 margin-left: -100%的理解](https://blog.csdn.net/darabiuz/article/details/122671084)
  
  对于该文章里的**对于margin取负值的理解**，我的看法是：因为中间容器设置了100%宽度，所以会把left容器和right容器挤到下边儿；中间元素，左边，右边都设置了浮动，
  
  其实我们可以假设他们就处于一条线上了，只是由于空间不够，所以左右元素被挤下来了
  
  由于html排列的顺序是 center left right，且都设置了浮动，但是因为空间不够，left和right被挤下来了，所以得靠margin设为负值来回到该在的地点
  
  > 当margin-left的值百分比时，是相对于父元素宽度的百分比
  
  https://zhuanlan.zhihu.com/p/103774667
  
  ```css
  使用flex布局：原理就是将容器设置为display: flex;两侧设置固定宽度,并且不允许弹性缩放flex: 0; flex-basis: 200px;中间允许弹性缩放，不设置宽度flex:1;
  ```
  
  





