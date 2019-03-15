
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [Template Engine 模板引擎](#template-engine-模板引擎)
* [Error Handling 错误处理](#error-handling-错误处理)
		* [Catching Errors](#catching-errors)
		* [The default error handler](#the-default-error-handler)
		* [Writing error handlers](#writing-error-handlers)

<!-- /code_chunk_output -->

# Template Engine 模板引擎
模板引擎允许你在应用程序中使用静态模板文件，模板引擎会将模板中的变量替换为实际的值，并将模板转化成HTML文件发送到客户端。
在使用模板引擎之前，需要设置如下内容：
* 模板视图：设置模板文件的位置。
```javascript
app.set('views', './views')
```
* 模板引擎：设置使用哪一种引擎渲染模板。
```javascript
app.set('view engine', 'pug')
```
使用不同的模板引擎需要在NPM中安装，Express默认自带的模板引擎是Pug。

在views目录下创建一个index.pug的模板文件：
```html
html
  head
    title= title
  body
    h1= message
```
然后在路由中渲染index.pug文件，
```javascript
app.get('/', function (req, res) {
  res.render('index', { title: 'Hey', message: 'Hello there!' })
})
```
当你请求'/'路径时，index.pug文件会被渲染成HTML发送到客户端。
每一次请求都会重新绘制HTML页面，模板引擎并不会缓存绘制好的HTML页面，如果有需要，用户需要自己缓存该页面。

# Error Handling 错误处理
### Catching Errors
Express的路由和中间件发生的同步错误不需要额外的操作，express会自动捕获并处理该错误。
```javascript {.line-numbers}
app.get("/", function (req, res) {
  throw new Error("BROKEN");
});

// Express捕获的错误如下：
Error: BROKEN
    at C:\Users\Kilin\Desktop\NodeTest\kilin.js:21:9
    at Layer.handle [as handle_request] (C:\Users\Kilin\Desktop\NodeTest\node_modules\express\lib\router\layer.js:95:5)
    at next (C:\Users\Kilin\Desktop\NodeTest\node_modules\express\lib\router\route.js:137:13)
    at Route.dispatch (C:\Users\Kilin\Desktop\NodeTest\node_modules\express\lib\router\route.js:112:3)
    at Layer.handle [as handle_request] (C:\Users\Kilin\Desktop\NodeTest\node_modules\express\lib\router\layer.js:95:5)
    at C:\Users\Kilin\Desktop\NodeTest\node_modules\express\lib\router\index.js:281:22
    at Function.process_params (C:\Users\Kilin\Desktop\NodeTest\node_modules\express\lib\router\index.js:335:12)
    at next (C:\Users\Kilin\Desktop\NodeTest\node_modules\express\lib\router\index.js:275:10)
    at expressInit (C:\Users\Kilin\Desktop\NodeTest\node_modules\express\lib\middleware\init.js:40:5)
    at Layer.handle [as handle_request] (C:\Users\Kilin\Desktop\NodeTest\node_modules\express\lib\router\layer.js:95:5)
```

Express的路由和中间件发生的异步错误需要传递给next(error)方法，然后express才会处理该错误。
```javascript {.line-numbers}
app.get("/", function(req, res, next) {
  fs.readFile("./file-does-not-exist", function(err, data) {
    if (err) {
      next(err);
    } else {
      res.send(data);
    }
  });
});

// Express捕获的错误如下：
Error: ENOENT: no such file or directory, open 'C:\Users\Kilin\Desktop\NodeTest\file-does-not-exist'
```

尽量使用Promise代替try...catch来捕获异步错误。

### The default error handler
Express有一个默认的的内置错误处理中间件，处于中间件栈的尾部。如果一个错误事件没有被其他中间件处理，那么内置的错误处理中间件会捕获它，并将该错误写入堆栈轨迹中（堆栈轨迹仅在开发环境有效）。

### Writing error handlers
错误处理中间件与其他的中间件一样，只是多了一个error参数。
```javascript {.line-numbers}
app.use(function (err, req, res, next) {
  console.error(err.stack)
  res.status(500).send('Something broke!')
})
```