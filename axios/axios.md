[toc]

### 二.XHR 的 [ajax](https://so.csdn.net/so/search?q=ajax&spm=1001.2101.3001.7020) 封装 (简单版axios)

#### 1.特点

1. 函数的返回值为`promise`, 成功的结果为`response`, 失败的结果为`error`

2. 能处理多种类型的请求: GET/POST/PUT/DELETE

3. 函数的参数为一个配置对象

   ```javascript
   {
   	url: '', // 请求地址
   	method: '', // 请求方式GET/POST/PUT/DELETE
   	params: {}, // GET/DELETE 请求的 query 参数
   	data: {}, // POST/PUT 请求的请求体参数
   }
   ```

   

4. 响应 json数据自动解析为js的对象/数组

#### 2.使用测试

```javascript
 <script>
        //获取按钮
        const btns = document.querySelectorAll('button');
        //第一个
        btns[0].onclick = function(){
            //发送 AJAX 请求
           axios({
               //请求类型
               method: 'GET',
               //url
               url:'http://localhost:3000/posts/2'
           }).then(response =>{
               console.log(response);
           })
        }

        //添加一篇新的文章
        btns[1].onclick = function(){
            //发送 AJAX 请求
            axios({
                //请求类型
                method: 'POST',
                //URL
                url: 'http://localhost:3000/posts',
                //设置请求体
                data: {
                    title: "今天天气不错, 还挺风和日丽的",
                    author: "张三"
                }
            }).then(response => {
                console.log(response);
            });
        }

        //更新数据
        btns[2].onclick = function(){
            //发送 AJAX 请求
            axios({
                //请求类型
                method: 'PUT',
                //URL
                url: 'http://localhost:3000/posts/3',
                //设置请求体
                data: {
                    title: "今天天气不错, 还挺风和日丽的",
                    author: "cyk"
                }
            }).then(response => {
                console.log(response);
            });
        }

        //删除数据
        btns[3].onclick = function(){
            //发送 AJAX 请求
            axios({
                //请求类型
                method: 'delete',
                //URL
                url: 'http://localhost:3000/posts/3',
            }).then(response => {
                console.log(response);
            });
        }
    </script>
```





 ### 三.axios的理解与使用

#### 1.axios是什么

1. 前端最流行的 ajax请求库
2. react/vue 官方都推荐使用 axios 发ajax 请求
3. [文档: https://github.com/axios/axios](https://github.com/axios/axios)

#### 2.axios 特点

1. 基于 xhr + promise 的异步 ajax请求库
2. 浏览器端/node 端都可以使用
3. 支持请求／响应拦截器
4. 支持请求取消
5. 请求/响应数据转换
6. 批量发送多个请求

#### 3.axios常用语法

`axios(config)`: 通用/最本质的发任意类型请求的方式
`axios(url[,config])`: 可以只指定url 发get 请求
`axios.request(config)`: 等同于axios(config)

`axios.get(url[,config])`: 发get 请求
`axios.delete(url[,config])`: 发delete 请求
`axios.post(url[,data,config])`: 发post 请求

`axios.put(url[,data,config])`: 发put 请求



`axios.defaults.xxx`: 请求的默认全局配置（method\baseURL\params\timeout…）

`axios.interceptors.request.use()`: 添加请求[拦截器](https://so.csdn.net/so/search?q=拦截器&spm=1001.2101.3001.7020)
`axios.interceptors.response.use()`: 添加响应拦截器



`axios.create([config])`: 创建一个新的axios(它没有下面的功能)



`axios.Cancel()`: 用于创建取消请求的错误对象
`axios.CancelToken()`: 用于创建取消请求的 token 对象
`axios.isCancel()`: 是否是一个取消请求的错误

`axios.all(promises)`: 用于批量执行多个异步请求
`axios.spread()`: 用来指定接收所有成功数据的回调函数的方法

<img src="/Users/ethereal_/Desktop/works/notes/图/20210304172721643.jpg" alt="20210304172721643" style="zoom:80%;" align="left" />

#### 4.难点语法的理解和使用

##### 1.`axios.create(config)`

1.根据指定配置创建一个新的axios, 也就是每个新axios都有自己的配置
2.新axios只是没有取消请求和批量发请求的方法, 其它所有语法都是一致的
3.为什么要设计这个语法?
(1) 需求: 项目中有部分接口需要的配置与另一部分接口需要的配置不太一样, 如何处理（比如有多个baseURL需要指定）
(2) 解决: 创建2个新axios, 每个都有自己特有的配置, 分别应用到不同要求的接口请求中

```javascript
const instance = axios.create({ // instance是函数类型
	baseURL: 'http://localhost:3000'
})
// 使用instance发Ajax请求
instance({
	url: '/posts'
})
instance.get('/posts')
```



##### 2.拦截器函数/ajax请求/请求的回调函数的调用顺序

- 1.说明: 调用axios()并不是立即发送ajax请求, 而是需要经历一个较长的流程
- 2.流程: 请求拦截器2 => 请求拦截器1 => 发ajax 请求 => 响应拦截器1 => 响应拦截器2 => 请求的回调
- 3.注意: 此流程是通过 promise 串连起来的, 请求拦截器传递的是config, 响应拦截器传递的是response

```javascript
 <script>
        // 基于Promise
        // 设置请求拦截器  config 配置对象
        axios.interceptors.request.use(function(config) {
            console.log('请求拦截器 成功 -1号');
            //可以修改config中的参数
            config.timeout=2000;
            return config;
        }, function (error){
            console.log('请求拦截器 失败 -1号');
            return Promise.reject(error);
        })

        axios.interceptors.request.use(function (config) {
            console.log('请求拦截器 成功 -2号');
            //修改 config 中的参数
            config.timeout = 2000;
            return config;
        }, function (error) {
            console.log('请求拦截器 失败 -2号');
            return Promise.reject(error);
        });

        // 设置响应拦截器
        axios.interceptors.response.use(function (response) {
            console.log('响应拦截器 成功 1号');
            return response.data;  //对响应结果进行处理
            // return response;
        }, function (error) {
            console.log('响应拦截器 失败 1号')
            return Promise.reject(error);
        });

        axios.interceptors.response.use(function (response) {
            console.log('响应拦截器 成功 2号')
            return response;
        }, function (error) {
            console.log('响应拦截器 失败 2号')
            return Promise.reject(error);
        });

        //发送请求
        axios({
            method: 'GET',
            url: 'http://localhost:3000/posts'
        }).then(response => {
            console.log('自定义回调处理成功的结果');
            console.log(response);   //与响应拦截器里的return response.data 对应，只会打印出相应的数据内容
        });
    </script>          
// request interceptor2 onResolved()
// request interceptor1 onResolved()
// response interceptor1 onResolved()
// response interceptor2 onResolved()
```



##### 3.取消请求

- 基本流程

  1.配置`cancelToken`对象
  2.缓存用于取消请求的`cancel`函数
  3.在后面特定时机调用`cancel`函数取消请求
  4.在错误回调中判断如果`error`是`cancel`,做相应处理

  - 基本实现

    **用express先搭建一个有延迟的服务器**

    ```javascript
    const express = require('express')
    const cors = require('cors')
    
    const app = express()
    
    // 使用cors, 允许跨域
    app.use(cors())
    // 能解析urlencode格式的post请求体参数
    app.use(express.urlencoded())
    // 能解析json格式的请求体参数
    app.use(express.json())
    
    app.get('/products1', (req, res) => {
      //设置延时
      setTimeout(() => {
        res.send([
          {id: 1, name: 'product1.1'},
          {id: 2, name: 'product1.2'},
          {id: 3, name: 'product1.3'}
        ])
      }, 1000 + Math.random()*2000);
      
    })
    
    app.get('/products2', (req, res) => {
      setTimeout(() => {
        res.send([{
            id: 1,
            name: 'product2.1'
          },
          {
            id: 2,
            name: 'product2.2'
          },
          {
            id: 3,
            name: 'product2.3'
          }
        ])
      }, 1000 + Math.random() * 2000);
    
    })
    
    app.listen(4000, () => {
      console.log('server app start on port 4000')
    })
    ```

    1.**点击按钮, 取消某个正在请求中的请求**

    ```javascript
    let cancel // 用于保存取消请求的函数
    function getProducts1() {
      axios({
        url: 'http://localhost:4000/products1',
        cancelToken: new axios.CancelToken(function executor(c){ // c是用于取消当前请求的函数
          // 保存取消函数，用于之后可能需要取消当前请求
          cancel = c;
        })
      }).then(
        response => {
          cancel = null
          console.log('请求1成功了', response.data)
        },
        error => {
          cancel = null
          console.log('请求1失败了', error.message, error) // 请求1失败了 强制取消请求 Cancel {message: "强制取消请求"}
        }
      )
    
    }
    
    function getProducts2() {
      axios({
          url: 'http://localhost:4000/products2'
      }).then(
        response => {
          console.log('请求2成功了', response.data)
        }
      )
    }
    
    function cancelReq() {
      // alert('取消请求')
      // 执行取消请求的函数
      if (typeof cancel === 'function'){
        cancel('强制取消请求')
      } else {
        console.log('没有可取消的请求')
      }
    }
    ```

    2.在请求一个接口前，取消前面一个未完成的请求

    ```javascript
    let cancel // 用于保存取消请求的函数
    function getProducts1() {
      // 在准备发请求前，取消未完成的请求
      if (typeof cancel === 'function'){
        cancel('取消请求')
      }
      axios({
        url: 'http://localhost:4000/products1',
        cancelToken: new axios.CancelToken(function executor(c){ // c是用于取消当前请求的函数
          // 保存取消函数，用于之后可能需要取消当前请求
          cancel = c;
        })
      }).then(
        response => {
          cancel = null
          console.log('请求1成功了', response.data)
        },
        error => {
          if (axios.isCancel(error)){
            console.log('请求1取消的错误', error.message)
          }else{ // 请求出错了
            cancel = null
            console.log('请求1失败了', error.message, error) // 请求1失败了 强制取消请求 Cancel {message: "强制取消请求"}
          }
        }
      )
    }
    
    function getProducts2() {
    // 在准备发请求前，取消未完成的请求
      if (typeof cancel === 'function'){
        cancel('取消请求')
      }
      axios({
          url: 'http://localhost:4000/products2',
          cancelToken: new axios.CancelToken(function executor(c){ 
          cancel = c;
        })
      }).then(
        response => {
          cancel = null
          console.log('请求2成功了', response.data)
        },
        error => {
          if (axios.isCancel(error)){
            console.log('请求2取消的错误', error.message)
          }else{ 
            cancel = null
            console.log('请求2失败了', error.message, error) 
          }
        }
      )
    }
    function cancelReq() {
      // alert('取消请求')
      // 执行取消请求的函数
      if (typeof cancel === 'function'){
        cancel('强制取消请求')
      } else {
        console.log('没有可取消的请求')
      }
    }
    ```

    





