---
title: express中间件
date: 2019-07-08 14:43:50
tags:
  - express
categories:
  - nodejs
---
Express应用程序本质上是一系列中间件函数调用。   
中间件可以做下面这些事情
- 执行任何代码
- 修改`request`和`response`对象
- 结束请求（响应周期）
- 调用堆栈中的下一个中间件函数  
如果当前中间件没有结束请求，那么需要调用`next()`,将控制权传递给下一个中间件函数，否则请求会被挂起，后边定义的中间件将不会执行

<!-- more -->

``` js
app.use(function (req, res, next) {
    console.log('first')
    // next()
})
app.post('/info', (req, res) => {
    console.log('****POST*****')
    console.log(req.body)
    res.status(200).send(req.body)
})
```
使用`postman`模拟请求`http://localhost:3002/info`,请求会在第一个中间件挂起，不会执行到第二个  
![有next](/images/middle-next.png)  
![没有next](/images/middle-no-next.png)  
`express`将中间件等信息存放在`stack`中，如果没有匹配到合适的中间件或者没有结束请求，那么就会循环往下匹配，直到最后一个。
express可以使用下面这些类型的中间件：
- `Application-level`
- `Router-level`
- `Built-in` 内置中间件
- `Error-handling`错误处理中间件
- `Third-party`第三方中间件

### Application-level 中间件
使用`app.use()`和`app.METHOD()`将中间件绑定到app对象。`METHOD`为`get,put、post`等
``` js
执行匹配任何路径（这里省略/）
app.use(function(req,res,next)=>{
    //do something
})
执行匹配/user路径的任何http请求
app.use('/user',(req,res,next)=>{
    //do something
})
执行匹配/user/:id路径的get请求
app.get('/user/:id',(req,res,next)=>{
    //do something
})
```
### Router-level 中间件
和`Application-level`工作方式类似，但是它是绑定在`express.Router()`上的
### Error-handling 中间件
定义方法差不多，只是它接收四个参数，使用相关方法如下
``` js
var express = require('express')
var bodyParser = require('body-parser')
var app = express()
var port = '3002'
app.use(bodyParser.json())
app.use(function (req, res, next) {
    console.log('first')
    next()
})
app.post('/info', (req, res) => {
    console.log('****POST*****')
    console.log(req.body)
    throw new Error('测试')
    // res.status(200).send(req.body)
})

app.use(function (err, req, res, next) {
    console.error(err)
    res.status(500).send(err.message)
})
app.listen(port, () => {
    console.log(`localhost:${port}`)
})
```
### 内置中间件
包含在`express`中的中间件功能
内置的中间件有：
- `express.static`提供静态资源
``` js
// 将静态文件目录设置为项目根目录+/public
app.use(express.static(__dirname + '/public'));
```
- `express.json`使用JSON解析传入的请求(express 4.16.0+有效)  
类似`body-parse`中的`bodyParse.json()`
``` js
app.use(express.json())
```
- `express.urlencoded`使用URL-encoded解析传入的请求(express 4.16.0+有效)  
类似`body-parse`中`bodyParser.urlencoded()`
```
app.use(express.urlencoded({extended:false}))
```
### 第三方中间件
需要安装模块获取需要的功能，然后在`application level`或者`router level`中加载到应用程序中,例如下面的增加`body-parser`
``` js
var express = require('express')
var bodyParser = require('body-parser')
var app = express()
app.use(bodyParser.json())
```

> 中间件next源码解析链接 https://www.cnblogs.com/aishangliming/p/6115359.html 