
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [Template Engine 模板引擎](#template-engine-模板引擎)

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