
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [同源策略](#同源策略)
	* [1. 同源策略](#1-同源策略)
	* [2. 为什么需要同源策略](#2-为什么需要同源策略)
	* [3. 不受同源策略限制的行为](#3-不受同源策略限制的行为)
	* [参考链接](#参考链接)

<!-- /code_chunk_output -->

# Same-Origin Policy 同源策略

## 1. 同源策略
URL由协议，域名，端口和路径组成。如果两个URL的协议，域名和端口都一样，则认为它们是同源的，反之则认为它们是不同源。
```javascript
http://mySite.com:80/index.html
```
上面这个URL中：
协议: http
域名: mySite.com
端口: 80 
路径: index.html

进行同源判断：
```javascript
http://mySite.com:80/path1/home.html    //同源，只有路径不一致
https://mySite.com:80/index.html        //非同源，协议不一致
http://xxx.mySite.com:80/index.html     //非同源，域名不一致
http://xxx.mySite.com:8080/index.html   //非同源，端口不一致
```
浏览器采用同源策略，禁止页面加载或执行与自身来源不同的域的任何脚本，这是一个用于隔离潜在恶意文件的重要安全机制。

## 2. 为什么需要同源策略
如果浏览器没有同源策略，会存在什么样的安全问题呢。下面从 DOM 同源策略和 XMLHttpRequest 同源策略来举例说明：

1. **如果没有 DOM 同源策略，也就是说不同域的 iframe 之间可以相互访问，那么黑客可以这样进行攻击**：
* 做一个假网站，里面用 iframe 嵌套一个银行网站 http://mybank.com。
* 把 iframe 宽高啥的调整到页面全部，这样用户进来除了域名，别的部分和银行的网站没有任何差别。
* 这时如果用户输入账号密码，我们的主网站可以跨域访问到 http://mybank.com 的 dom 节点，就可以拿到用户的账户密码了。

2. **如果 XMLHttpRequest 同源策略，那么黑客可以进行 CSRF（跨站请求伪造） 攻击**：
* 用户登录了自己的银行页面 http://mybank.com，http://mybank.com 向用户的 cookie 中添加用户标识。
* 用户浏览了恶意页面 http://evil.com，执行了页面中的恶意 AJAX 请求代码。
* http://evil.com 向 http://mybank.com 发起 AJAX HTTP 请求，请求会默认把 http://mybank.com 对应 cookie 也同时发送过去。
* 银行页面从发送的 cookie 中提取用户标识，验证用户无误，response 中返回请求数据。此时数据就泄露了。
* 而且由于 Ajax 在后台执行，用户无法感知这一过程。

因此，有了浏览器同源策略，我们才能更安全的上网。

## 3. 不受同源策略限制的行为
* 页面中的链接，重定向和表单提交是不受同源策略影响的。
* 页面当中的`<script>、<img>、<iframe>和<link>`不受同源策略影响。

对于一段JavaScript脚本来说，其源与它存储的地址无关，而取决于脚本被加载的页面。
```javascript
<sricpt src="http://siteA.com/scripts/a.js"></sricpt>
<sricpt src="https://siteB.com/scripts/b.js"></sricpt>
```
在同一个页面加载上面两个脚本，它们均被认为与当前页面同源。除了`<script>`标签，HTML还具有其它一些具有src属性的标签（比如`<img>、<iframe>和<link>`等），它们均具有跨域加载资源的能力，所以同源策略对它们不做限制。对于这些具有src属性的HTML标签来说，标签的每次加载都意味着针对目标地址的一次HTTP-GET请求。

同源策略主要限制了通过XMLHttpRequest实现的Ajax请求，如果请求的是一个“非同源”地址，浏览器将不允许读取返回的内容。

## 参考链接
1. https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy
2. https://www.cnblogs.com/artech/p/cors-4-asp-net-web-api-01.html