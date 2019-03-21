
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [HTTP Strict-Transport-Security](#http-strict-transport-security)
	* [1. 为什么要有HSTS](#1-为什么要有hsts)
	* [2. 什么是HSTS](#2-什么是hsts)
	* [3. HSTS语法](#3-hsts语法)
	* [4. HSTS不足](#4-hsts不足)
	* [5. 总结](#5-总结)
	* [参考链接](#参考链接)

<!-- /code_chunk_output -->

# HTTP Strict-Transport-Security
## 1. 为什么要有HSTS
在互联网早期，一般使用HTTP协议来传输数据，由于HTTP是明文传输信息，没有任何加密，这样的访问方式并不十分安全，因为在第一次HTTP请求和重定向的时候，攻击者可以以中间人的方式劫持这次请求，从而完成中间人攻击。

解决办法：使用HSTS。

## 2. 什么是HSTS
HSTS是一种web安全策略机制。该策略的核心是一个HTTP响应头，通过这个响应同，让浏览器在一段时间内只能通过HTTPS进行访问该网站，并且在浏览器发现当前连接不安全的情况下拒绝用户的后续访问。

当我们输入网站地址baidu.com后，浏览器会自动将地址变更为`http://baidu.com`，然后发送给服务器，服务器接收请求后发现不是一个https类型的连接，服务器返回给浏览器一个带有https类型链接的地址和一个状态码3xx，浏览器根据状态码和重定向地址访问`https://baidu.com`，服务器与浏览器建立链接，并返回一个响应，此时响应头中含有一个Strict-Transport-Security字段，表明该网站需要进行https类型访问，这样浏览器在以后的每次请求发出之前，都会自动将用户输入的内容转化为https开头的地址。

## 3. HSTS语法
Strict-Transport-Security: max-age=<expire-time>
Strict-Transport-Security: max-age=<expire-time>; includeSubDomains
Strict-Transport-Security: max-age=<expire-time>; preload

max-age:设置在浏览器收到这个请求后的<expire-time>秒的时间内凡是访问这个域名下的请求都使用HTTPS请求。
includeSubDomains:如果这个可选的参数被指定，那么说明此规则也适用于该网站的所有子域名。
preload:查看 Preloading Strict Transport Security 获得详情。不是标准的一部分。

比如，`https://xxx.com` 的响应头含有Strict-Transport-Security: max-age=31536000; includeSubDomains。这意味着两点：
* 在接下来的一年（即31536000秒）中，浏览器只要向xxx或其子域名发送HTTP请求时，必须采用HTTPS来发起连接。比如，用户点击超链接或在地址栏输入 `http://xxx/` ，浏览器应当自动将 http 转写成 https，然后直接向 `https://xxx/` 发送请求。
* 在接下来的一年中，如果 xxx 服务器发送的TLS证书无效，用户不能忽略浏览器警告继续访问网站

服务器开启HSTS的方法是，当客户端通过HTTPS发出请求时，在服务器返回的超文本传输协议响应头中包含Strict-Transport-Security字段。非加密传输时设置的HSTS字段无效。

## 4. HSTS不足
HSTS存在一个比较薄弱的环节，那就是浏览器没有当前网站的HSTS信息的时候，或者第一次访问网站的时候，依然需要一次明文的HTTP请求和重定向才能切换到HTTPS，以及刷新HSTS信息。而就是这么一瞬间却给攻击者留下了可乘之机，使得他们可以把这一次的HTTP请求劫持下来，继续中间人攻击。

针对上面的攻击，HSTS也有应对办法，那就是在浏览器里内置一个列表(preload list)，只要是在这个列表里的域名，无论何时、何种情况，浏览器都只使用HTTPS发起连接。这个列表由Google Chromium维护，FireFox、Safari、IE等主流浏览器均在使用。

## 5. 总结
现代网站建设应该尽量使用HTTPS类型的安全链接，通过HSTS的设置，可以让浏览器自动将HTTP类型的链接转化成HTTPS类型，HSTS必须配置的是max-age，HSTS不足是浏览器第一次与网站建立链接之前，可能会被劫持。
HSTS的逻辑流程如下图：
![HSTS逻辑流程图](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/security/Secutiry_HTTP_Headers_HSTS_1.png?raw=true)

## 参考链接
1. https://www.fxw.la/news/8137.html
2. https://blog.risingstack.com/node-js-security-checklist/
3. https://developer.mozilla.org/zh-CN/docs/Security/HTTP_Strict_Transport_Security