[toc]

### 浏览器

#### 跨域是什么？如何解决跨域问题？

**跨域**：当前页面中的某个接口请求的地址和当前页面的地址如果**协议、域名、端口**其中有一项不同，就说该接口跨域了。

跨域限制的**原因**：浏览器为了保证网页的安全，出的同源协议策略。

解决方案：

- **CORS**：跨域资源共享，目前最常用的一种解决办法，通过设置后端允许跨域实现。
  res.setHeader('Access-Control-Allow-Origin', '*');
- **JSONP**：利用的原理是script标签可以跨域请求资源，将回调函数作为参数拼接在url中。后端收到请求，调用该回调函数，并将数据作为参数返回去，注意设置响应头返回文档类型，应该设置成javascript。
- **Proxy**:配置代理

#### 浏览器输入url都发生了啥

1、浏览器的地址栏输入URL并按下回车。

2、浏览器查找当前URL是否存在缓存，并比较缓存是否过期。

3、DNS解析URL对应的IP。

4、根据IP建立TCP连接（三次握手）。

5、HTTP发起请求。

6、服务器处理请求，浏览器接收HTTP响应。

7、渲染页面，构建DOM树。

8、关闭TCP连接（四次挥手）。

##### 浏览器内核

在浏览器中有一个最重要的模块，它主要的作用是把一切`请求回来的资源变成可视化的图像`，这个模块就是`浏览器内核`，通常它也被称为`渲染引擎`。

1、IE：`Trident`

2、Safari：`WebKit`。WebKit本身主要是由两个小引擎构成的，一个正是渲染引擎“WebCore”，另一个则是javascript解释引擎“JSCore”，它们均是从KDE的渲染引擎KHTML及javascript解释引擎KJS衍生而来。

3、Chrome：`Blink`。在13年发布的Chrome 28.0.1469.0版本开始，Chrome放弃Chromium引擎转而使用最新的Blink引擎（基于WebKit2——苹果公司于2010年推出的新的WebKit引擎），Blink对比上一代的引擎精简了代码、改善了DOM框架，也提升了安全性。

4、Opera：2013年2月宣布放弃`Presto`，采用`Chromium`引擎，又转为`Blink`引擎

5、Firefox：`Gecko`。



#### 前端路由hash模式和history模式

##### hash模式

url带#号，发送http请求时不会带上#号，改变hash的时候不会重新加载页面

兼容性好

##### history模式

url地址美观，路由跳转不需要重新加载页面。

兼容性相较于hash较差

部署到服务器时需要后端人员支持，解决刷新页面服务端404的问题

#### 页面渲染流程

>1．解析HTML文件，创建DOM树
>
>2．解析CSS，形成CSS对象模型
>
>3．将CSS与DOM合并，构建渲染树（renderingtree）
>
>4．回流与重绘  （重排 重绘)
>
>5．Display 

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20220720210222290.png" alt="image-20220720210222290" style="zoom:50%;" />





#### 回流(重排)与重绘

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



### HTTP

#### HTTP常用状态码

- 1XX 表示消息
- 2XX 表示成功
- 3XX 表示重定向
- 4XX 表示客户端错误
- 5XX 表示服务端错误

| 状态码 | 状态码英文名称        | 中文描述                                                     |
| :----- | :-------------------- | :----------------------------------------------------------- |
| 200    | OK                    | 请求成功。一般用于GET与POST请求                              |
|        |                       |                                                              |
| 301    | Moved Permanently     | 永久重定向。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替 |
| 302    | Found                 | 临时重定向。与301类似。但资源只是临时被移动。客户端应继续使用原有URI |
| 304    | Not Modified          | 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源 |
|        |                       |                                                              |
| 400    | Bad Request           | 客户端请求的语法错误，服务器无法理解                         |
| 401    | Unauthorized          | 未被授权，请求要求用户的身份认证                             |
| 403    | Forbidden             | 服务器理解请求客户端的请求，但是拒绝执行此请求               |
| 404    | Not Found             | 服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页 |
|        |                       |                                                              |
| 500    | Internal Server Error | 服务器内部错误，无法完成请求                                 |
| 502    | Bad Gateway           | 作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应 |
| 503    | Service Unavailable   | 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中 |



#### HTTP的缓存机制

[神说要有光 HTTP 的缓存为什么这么设计？](https://mp.weixin.qq.com/s/0P8_lnVf2_zMzIBJ20qajA)

[神说要有光 面试官：你懂 HTTP 缓存，那说下浏览器强制刷新是怎么实现的](https://juejin.cn/post/7163506251304239135)

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20220520123934118.png" alt="image-20220520123934118" style="zoom:35%;" />

##### 强缓存

强缓存两个相关字段，**「Expires」**，**「Cache-Control」**。

**「强缓存分为两种情况，一种是发送HTTP请求，一种不需要发送。」**

首先检查强缓存，这个阶段**不需要发送HTTP请求。**通过查找不同的字段来进行，不同的HTTP版本所以不同。

- HTTP1.0版本，使用的是Expires，HTTP1.1使用的是Cache-Control



**Expires**

`Expires`即过期时间，时间是相对于服务器的时间而言的，存在于服务端返回的响应头中，在这个过期时间之前可以直接从缓存里面获取数据，无需再次请求。比如下面这样:

```
Expires:Mon, 29 Jun 2020 11:10:23 GMT
```

表示该资源在2020年7月29日11:10:23过期，过期时就会重新向服务器发起请求。

这个方式有一个问题：**「服务器的时间和浏览器的时间可能并不一致」**，所以HTTP1.1提出新的字段代替它。



**Cache-Control**

HTTP1.1版本中，使用的就是该字段，这个字段采用的时间是过期时长，对应的是max-age。

```
Cache-Control:max-age=6000
```

上面代表该资源返回后6000秒，可以直接使用缓存。

当然了，它还有其他很多关键的指令，梳理了几个重要的👇

注意点：

- 当Expires和Cache-Control同时存在时，优先考虑Cache-Control。
- 当然了，当缓存资源失效了，也就是没有命中强缓存，接下来就进入协商缓存👇

##### 协商缓存

强缓存失效后，浏览器在请求头中携带响应的`缓存Tag`来向服务器发送请求，服务器根据对应的tag，来决定是否使用缓存。

缓存分为两种，**「Last-Modified」** 和 **「ETag」**。两者各有优势，并不存在谁对谁有`绝对的优势`，与上面所讲的强缓存两个Tag所不同。



**Last-Modified**

这个字段表示的是**「最后修改时间」**。在浏览器第一次给服务器发送请求后，服务器会在响应头中加上这个字段。

浏览器接收到后，**「如果再次请求」**，会在请求头中携带`If-Modified-Since`字段，这个字段的值也就是服务器传来的最后修改时间。

服务器拿到请求头中的`If-Modified-Since`的字段后，其实会和这个服务器中`该资源的最后修改时间`对比:

- 如果请求头中的这个值小于最后修改时间，说明是时候更新了。返回新的资源，跟常规的HTTP请求响应的流程一样。
- 否则返回304，告诉浏览器直接使用缓存。



**ETag**

ETag是服务器根据当前文件的内容，对文件生成唯一的标识，比如MD5算法，只要里面的内容有改动，这个值就会修改，服务器通过把响应头把该字段给浏览器。

浏览器接受到ETag值，会在下次请求的时候，将这个值作为`If-None-Match`这个字段的内容，发给服务器。

服务器接收到`If-None-Match`后，会跟服务器上该资源的**「ETag」**进行比对👇

- 如果两者一样的话，直接返回304，告诉浏览器直接使用缓存

- 如果不一样的话，说明内容更新了，返回新的资源，跟常规的HTTP请求响应的流程一样

  

**两者对比**

- 性能上，`Last-Modified`优于`ETag`，`Last-Modified`记录的是时间点，而`Etag`需要根据文件的MD5算法生成对应的hash值。

- 精度上，ETag优于Last-Modified，ETag按照内容给资源带上标识，能准确感知资源变化，Last-Modified

  在某些场景并不能准确感知变化，比如👇

  - 编辑了资源文件，但是文件内容并没有更改，这样也会造成缓存失效。
  - Last-Modified 能够感知的单位时间是秒，如果文件在 1 秒内改变了多次，那么这时候的 Last-Modified 并没有体现出修改了。

最后，**「如果两种方式都支持的话，服务器会优先考虑`ETag`」**。



#### cookie sessionStorage localStorage 

>**数据存储位置、生命周期、存储大小、写入方式、数据共享、发送请求时是否携带、应用场景**

**相同点**：都是存储在浏览器本地

**不同点**:1.cookie是由服务端写入的，sessionStorage和localStorage是前端写入的

​      2.cookie的生命周期是由服务器端在写入的时候就设置好的，SessionStorage是页面关闭的时候就会自动清除.LocalStorage是写入就一直存在，除非手动清除.

​      3.cookie的存储空间比较小大概4KB，SessionStorage、 LocalStorage存储空间比较大，大概5M。

​      4.Cookie、SessionStorage、 LocalStorage数据共享都遵循同源原则，SessionStorage还限制必须是同一个页面。

​      5.在前端给后端发送请求的时候会自动携带Cookie中的数据，但是SessionStorage、LocalStorage不会，不过可以手动携带。

**应用场景**：由于它们的以上区别，所以它们的应用场景也不同，Cookie一般用于存储登录验证信息SessionID或者token，LocalStorage常用于存储不易变动的数

据，减轻服务器的压力，SessionStorage可以用来检测用户是否是刷新进入页面，如音乐播放器恢复播放进度条的功能。

`localStorage` 最主要的特点是：

- 在**同源**的所有标签页和窗口之间共享数据。
- 数据不会过期。它在浏览器重启甚至系统重启后仍然存在。

`sessionStorage`的数据只存在于当前浏览器标签页。

- 具有相同页面的另一个标签页中将会有不同的存储。
- 但是，它在同一标签页下的 **iframe** 之间是共享的（假如它们来自相同的源）。

- 数据在页面刷新后仍然保留，但在关闭/重新打开浏览器标签页后不会被保留。

[傻傻分不清之 Cookie、Session、Token、JWT](https://juejin.cn/post/6844904034181070861#heading-18)

### HTML

#### HTML语义化

>1、易于用户阅读，css样式丢失的时候能让页面呈现清晰的结构。
>
>2、有利于SEO，搜索引擎根据标签来确定上下文和各个关键字的权重。
>
>3、方便其他设备解析，如盲人阅读器根据语义渲染网页
>
>4、有利于开发和维护，语义化更具可读性，代码更好维护，与CSS3关系更和谐  



#### HTML新特性

一、语义标签

二、增强型表单

三、视频和音频

四、Canvas绘图

五、SVG绘图

六、地理定位

七、拖放API

八、WebWorker

九、WebStorage

十、WebSocket



### CSS

#### link与@import区别

(1) link属于HTML标签，而@import是CSS提供的; 

(2) 页面被加载的时，link会同时被加载，而@import引用的CSS会等到页面被加载完再加载; 

(3) import只在IE5以上才能识别，而link是HTML标签，无兼容问题;

##### CSS阻塞情况以及优化

1、style标签中的样式：由`HTML解析器`进行解析，`不会`阻塞浏览器渲染(可能会产生“闪屏现象”)，不会阻塞DOM解析

2、link引入的CSS样式：由`CSS解析器`进行解析，`会`阻塞浏览器渲染，会阻塞后面的js语句执行，不阻塞DOM的解析

3、优化：使用CDN节点进行外部资源加速，对CSS进行压缩，优化CSS代码(不要使用太多层选择器)。

#### 为什么CSS要放在头部，js要放在body底部

>因为css是要在网页渲染外观的时候就要调用的，所以要预先调入内存，因此放在头部。
>
>而js因为有可能需要调用网页DOM结构中的元素，所以必须等HTML的整个DOM结构都调入内存后才开始运行（否则就很可能因为无法找到元素而出错），所以要放在底部。
>
>但是最好或者说最标准的用法是把js代码放入window对象的onload事件中，这样就可以把整个js代码放到网页的任何位置了（包括头部）



#### 清除浮动

```css
1.开启BFC
2.clear:both清除浮动元素对当前元素所产生的影响
3.使用after before伪元素清除浮动
```



#### BFC块级格式化上下文

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





#### Flex

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
-  `flex-basis: 0%` ：该属性定义在分配多余空间之前，元素占据的主轴空间。浏览器就是根据这个属性来**计算是否有多余空间**的。默认值为 `auto` ，即项目本身大小。设置为 `0%` 之后，因为有 `flex-grow` 和 `flex-shrink` 的设置会自动放大或缩小。在做两栏布局时，如果右边的自适应元素 `flex-basis` 设为 `auto` 的话，其本身大小将会是 `0` 。





#### 一个盒子垂直水平居中有哪些方法？

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



#### 自适应布局

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
  





