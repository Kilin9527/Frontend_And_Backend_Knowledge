<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [一、Express的中间件](#一-express的中间件)
* [二、Express的中间件类型](#二-express的中间件类型)
	* [1. Application-level middleware](#1-application-level-middleware)
	* [2. Router-level middleware](#2-router-level-middleware)
	* [3. Error-handling middleware](#3-error-handling-middleware)
	* [4. Built-in middleware](#4-built-in-middleware)
	* [5. Third-party middleware](#5-third-party-middleware)

<!-- /code_chunk_output -->

# 一、Express的中间件
中间件是一个可以访问请求对象(request object)，响应对象(response object)和next方法的函数。

中间件可以可以执行如下操作:
* 执行任何代码。
* 修改request和response
* 结束request-response循环
* 调用next中间件

如果当前中间件没有结束request-response循环，那么它必须调用next()，否则该请求会被挂起。

# 二、Express的中间件类型
Express分为如下五种类型：
* Application-level middleware
* Router-level middleware
* Error-handling middleware
* Built-in middleware
* Third-party middleware

## 1. Application-level middleware
通过app.use()和app.MOTHED()方法可以绑定一个应用程序级别的中间件，其中METHOD是HTTP Method中的一种。
例一：每次请求都会执行的中间件。
```javascript {.line-numbers}
var app = express()

app.use(function (req, res, next) {
  console.log('Time:', Date.now())
  next()
})
```

例二：每次请求/user/:id路径时都会执行的中间件，无论是什么样的HTTP Mothed。
```javascript {.line-numbers}
app.use('/user/:id', function (req, res, next) {
  console.log('Request Type:', req.method)
  next()
})
```

例三：每次GET请求，路径为/user/:id时执行的中间件。
```javascript {.line-numbers}
app.get('/user/:id', function (req, res, next) {
  res.send('USER')
})
```

例四：多个中间件。
```javascript {.line-numbers}
app.use('/user/:id', function (req, res, next) {
  console.log('Request URL:', req.originalUrl)
  next()
}, function (req, res, next) {
  console.log('Request Type:', req.method)
  next()
})
```

例五：同一个请求方法，同一个路由设置了两次处理中间件，第二次设置的永远不会执行，因为第一次的中间件处理函数通过res.send()方法已经结束了request-response循环。
```javascript {.line-numbers}
app.get('/user/:id', function (req, res, next) {
  console.log('ID:', req.params.id)
  next()
}, function (req, res, next) {
  res.send('User Info')
})

// handler for the /user/:id path, which prints the user ID
app.get('/user/:id', function (req, res, next) {
  res.end(req.params.id)
})
```

例六：如果想跳过当前路由，可以使用next('route')方法，当user/id是0时，服务器返回special，当user/id不为0时，服务器返回regular。
```javascript {.line-numbers}
app.get('/user/:id', function (req, res, next) {
  if (req.params.id === '0') next('route')
  else next()
}, function (req, res, next) {
  res.send('regular')
})

app.get('/user/:id', function (req, res, next) {
  res.send('special')
})
```

## 2. Router-level middleware
路由级别和应用程序级别的中间件的工作方式一样，只是需要绑定express.router()实例。
路由级别的中间件通过router.use()和router.METHOD()方法加载。

例一：下面代码是从应用程序级别的中间件复制，挪到路由级别的中间件上使用。
```javascript {.line-numbers}
var app = express()
var router = express.Router()

router.use(function (req, res, next) {
  console.log('Time:', Date.now())
  next()
})

router.use('/user/:id', function (req, res, next) {
  console.log('Request URL:', req.originalUrl)
  next()
}, function (req, res, next) {
  console.log('Request Type:', req.method)
  next()
})

router.get('/user/:id', function (req, res, next) {
  if (req.params.id === '0') next('route')
  else next()
}, function (req, res, next) {
  res.render('regular')
})

router.get('/user/:id', function (req, res, next) {
  console.log(req.params.id)
  res.render('special')
})

app.use('/admin', router)
```

例二：当用户访问/admin时，如果headers中有'x-auth'字段，程序运行到router.use()中，执行next操作，进入router.get()方法，返回'hello, user!'信息。如果headers中没有'x-auth'字段，则执行next('router')操作，进入到app.use()中的第二个中间件函数，返回401信息。
```javascript {.line-numbers}
var app = express()
var router = express.Router()

router.use(function (req, res, next) {
  if (!req.headers['x-auth']) return next('router')
  next()
})

router.get('/', function (req, res) {
  res.send('hello, user!')
})

app.use('/admin', router, function (req, res) {
  res.sendStatus(401)
})
```

## 3. Error-handling middleware
错误处理中间件必须有四个参数才能定义未错误处理中间件。定义错误处理中间件和定义其他中间件一样，只是函数参数多了一个error信息。
```javascript {.line-numbers}
app.use(function (err, req, res, next) {
  console.error(err.stack)
  res.status(500).send('Something broke!')
})
```

## 4. Built-in middleware
从Express4.0开始不再依赖于Connect框架，以前Express中包含的中间件现在以独立模块的形式提供。可以参看[Connect中间件列表](https://github.com/senchalabs/connect#middleware)。
Express本身有如下的内置中间件：
* **express.static**: 服务于静态文件，如Html文件，图片等.
* **express.json**: 使用JSON格式有效负载解析request。 NOTE: Available with Express 4.16.0+
* **express.urlencoded**: 使用URL-encoded格式有效负载解析request. NOTE: Available with Express 4.16.0+

## 5. Third-party middleware
使用第三方中间件，首先用NPM安装第三方中间件，然后在应用程序或者路由级别使用它。
```javascript
$ npm install cookie-parser
```

```javascript
var express = require('express')
var app = express()
var cookieParser = require('cookie-parser')

// load the cookie-parsing middleware
app.use(cookieParser())
```