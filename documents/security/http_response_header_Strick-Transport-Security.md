
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [HTTP -> Response Header -> Strict-Transport-Security](#http-response-header-strict-transport-security)
	* [1. Strict-Transport-Security语法](#1-strict-transport-security语法)
	* [2. Strict-Transport-Security使用方式](#2-strict-transport-security使用方式)
	* [3. Strict-Transport-Security产生背景](#3-strict-transport-security产生背景)
	* [4. Strict-Transport-Security作用原理](#4-strict-transport-security作用原理)
	* [5. Strict-Transport-Security不足](#5-strict-transport-security不足)
	* [6. 总结](#6-总结)
	* [参考链接](#参考链接)

<!-- /code_chunk_output -->

# HTTP -> Response Header -> Strict-Transport-Security
Strict-Transport-Security是一个HTTP响应头，该响应头的主要功能是告诉浏览器是否只能通过HTTPS方式与服务器通信。

## 1. Strict-Transport-Security语法
* Strict-Transport-Security: max-age=`<expire-time>`
* Strict-Transport-Security: max-age=`<expire-time>`; includeSubDomains
* Strict-Transport-Security: max-age=`<expire-time>`; preload

**max-age**:设置在浏览器收到这个请求后的`<expire-time>`秒的时间内凡是访问这个域名下的请求都使用HTTPS请求。
**includeSubDomains**:如果这个可选的参数被指定，那么说明此规则也适用于该网站的所有子域名。
**preload**:查看 Preloading Strict Transport Security 获得详情。不是标准的一部分。

比如，`http://mySite.com` 的响应头含有Strict-Transport-Security: max-age=31536000; includeSubDomains。这意味着两点：
* 在接下来的一年（即31536000秒）中，浏览器只要向mySite或其子域名发送HTTP请求时，必须采用HTTPS来发起连接。
* 在接下来的一年中，如果mySite服务器发送的TLS证书无效，用户不能忽略浏览器警告继续访问网站。

## 2. Strict-Transport-Security使用方式
在服务器的响应头中添加该字段并设置对应的值。
```javascript {.line-numbers}
// 伪代码
response.header.append({
    key: "Strict-Transport-Security", 
    value: "max-age=31536000; includeSubDomains"
})
```

## 3. Strict-Transport-Security产生背景
在互联网早期，一般使用HTTP协议来传输数据，由于HTTP是明文传输信息，没有任何加密，这样的访问方式并不十分安全，HTTP请求时可能会被中间人劫持，从而完成[中间人攻击](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/attack_manInTheMiddle.md)。

Strict-Transport-Security通过设置HTTP响应头，让浏览器在接下来的一段时间内只能通过HTTPS方式访问服务器，并且在浏览器发现当前连接不安全的情况下拒绝用户的后续访问。

## 4. Strict-Transport-Security作用原理
1. 用户访问网站 `baidu.com`。
2. 如果是第一次链接该网站，则浏览器自动将地址转变为`http://baidu.com`。
3. baidu服务器收到请求，认为HTTP链接不是一个安全的链接，返回一个重定向的状态码3XX和一个重定向地址`https://baidu.com`。
4. 浏览器收到服务器的返回，用重定向的网站继续访问`https://baidu.com`，链接建立，服务器返回给浏览器一个Response，该Response的Header中写入了Strict-Transport-Security信息。
5. 浏览器根据Strict-Transport-Security指定的时间，在以后每次访问baidu时，都会自动的将所有的非HTTPS类型请求自动转化为HTTPS类型的请求。

如下图所示：
![HSTS逻辑流程图](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/security/Secutiry_HTTP_Headers_HSTS_1.png?raw=true)

## 5. Strict-Transport-Security不足
Strict-Transport-Security存在一个比较薄弱的环节，那就是浏览器没有当前网站的Strict-Transport-Security信息的时候，或者第一次访问网站的时候，依然需要一次明文的HTTP请求和重定向才能切换到HTTPS，以及刷新Strict-Transport-Security信息。而就是这么一瞬间却给攻击者留下了可乘之机，使得他们可以把这一次的HTTP请求劫持下来，开启中间人攻击。

针对上面的攻击，也有应对办法，那就是在浏览器里内置一个列表(preload list)，只要是在这个列表里的域名，无论何时、何种情况，浏览器都只使用HTTPS发起连接。这个列表由Google Chromium维护，FireFox、Safari、IE等主流浏览器均在使用。

## 6. 总结
现代网站建设应该尽量使用HTTPS类型的安全链接。

通过Strict-Transport-Security的设置，可以让浏览器强制使用HTTPS类型的链接，从而提高安全性。

Strict-Transport-Security的不足是浏览器第一次与网站建立链接之前，可能会被劫持。

## 参考链接
1. https://www.fxw.la/news/8137.html
2. https://blog.risingstack.com/node-js-security-checklist/
3. https://developer.mozilla.org/zh-CN/docs/Security/HTTP_Strict_Transport_Security