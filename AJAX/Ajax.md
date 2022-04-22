[toc]

#### express的使用

`npm init --yes`初始化

`npm i express`安装express框架

> demo.js

```js
//引入express
const express=require("express");
//创建应用对象
const app=express();
//3.创建路由规则
//request是对请求报文的封装，response是对响应报文的封装
app.get('/',(request,response) => {
    //设置响应
    response.send("Hello,express");
});
//4.监听端口启动服务
app.listen(6000,() => {
    console.log("服务已经启动，6000端口监听中......");
});
```

`node demo.js`开启服务

#### 1.1 ajax简介

AJAX全称Asynchronous JavaScript And XML，就是异步的js和XML。

通过Ajax可以在浏览器中向服务器中发送异步请求，最大的优势是无需刷新获取数据

#### 1.2 XML简介

可扩展标记语言，全都是自定义标签，被设计用来存储和传输数据。

与HTML类似，不同的就是HTML中都是预定义标签，而XML中没有预定义标签。

#### 1.3 AJAX的特点

##### 优点 

①无需刷新页面即可与服务器端进行通信

②允许你根据用户事件来更新部分页面内容

##### 缺点

①没有浏览历史，不能回退

②存在跨域问题(同源)

③SEO不友好



#### 1.4 HTTP

超文本传输协议

##### 请求报文 

>行    POST  /s?ie=utf-8 HTTP/1.1
>
>头    Host:atguigu
>
>​        Cookie:name=guigu 
>
>​        Content-type:application/x-www-form-urlencoded
>
>​        User-agent:chrome 83
>
>空行
>
>体  username=admin&password=admin

get和post的区别在于，get请求没法传数据，单纯读取一个接口的资源，而post会传入一定的数据提供后台进行操作

##### 响应报文

>行    HTTP1.1    200(状态码)   OK                
>
>头    Content-Type：text/html
>
>​        Content-encoding:gzip 
>
>​        Content-type:application/x-www-form-urlencoded
>
>空行
>
>体    <html><header></header></html>

**响应状态码：**404(找不到)   403(被禁止)   401(未授权)  500(内部错误)  200(OK)

##### GET和POST

GET用于从指定资源请求数据，查询字符串（名称/值对）是在 GET 请求的 URL 中发送的：

~~~javascript
 //初始化，给第二个url参数发送get请求(设置请求方法和url)
 xhr.open("GET",'http://localhost:8000/server?cyk=200&xyh=300')
 //发送
 xhr.send()
~~~

POST用于将数据发送到服务器来创建/更新资源，通过 POST 发送到服务器的数据存储在 HTTP 请求的请求主体中

~~~javascript
Xhr.open("POST", "http://localhost:8000/server");
//post通过send来发送请求体
Xhr.send("cyk=100&xyh=200");
~~~





#### 2.1同源策略

同源策略（Same-Origin Policy）最早由 Netscape 公司提出，是浏览器的一种安全策略。

同源：协议、域名、端口号 必须完全相同

违背同源策略就是跨域

#### 2.2解决跨域

##### 2.2.1 JSONP

- JSONP是什么

JSONP (JSON with Padding)，是一个非官方的跨域解决方案，纯粹凭借程序员的聪明才智开发出来，只支持get请求

- JSONP 怎么工作的？

在网页有一些标签天生具有跨域能力，比如：img, link, iframe, script

JSONP就是利用**script**标签的跨域能力来发送请求的

- JSONP的使用

动态的创建一个script标签

```javascript
var script = document.createElement("script");
```

设置script的src，设置回调函数

```javascript
script.src = "http://locallhost:3000/textAJAX?callback=abc"
```

##### 2.2.2 cors

推荐阅读：

- http://www.ruanyifeng.com/blog/2016/04/cors.html
- https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS

1. CORS是什么？

   CORS (Cross-Origin Resource Sharing), 跨域资源共享。CORS 是官方的跨域解决方案，它的特点是不需要在客户端做任何特殊的操作，完全在服务器中进行处理，支持 get 和 post 等请求。跨域资源共享标准新增了一组 HTTP 首部字段（响应头），允许服务器声明哪些源站通过浏览器有权限访问哪些资源

2. CORS怎么工作的？

   CORS 是通过设置一个响应头来告诉浏览器，该请求允许跨域，浏览器收到该响应以后就会对响应放行。

3. CORS 的使用

   主要是服务端的设置：

   ```javascript
   rounter.get("/testAJAX",function(req, res){
   
   })
   ```