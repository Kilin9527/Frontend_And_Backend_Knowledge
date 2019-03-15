
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [HTTP Strict-Transport-Security](#http-strict-transport-security)
	* [1. 为什么要有HSTS](#1-为什么要有hsts)
			* [HSTS作用](#hsts作用)
			* [HSTS不足](#hsts不足)
		* [参考链接](#参考链接)

<!-- /code_chunk_output -->

# HTTP Strict-Transport-Security
## 1. 为什么要有HSTS
在互联网早期，一般使用HTTP协议来传输数据，由于HTTP是明文传输信息，没有任何加密，所以安全性极低，后来HTTPS协议诞生，数据通过HTTPS传输会进行加密，安全性有所提高，现代大分部网站，都是以HTTPS为标准开发。
当我们输入一个网站`www.example.com`，该网址会被转化成`http://www.example.com`格式访问服务器，服务器认为这样的访问方式不安全，将页面重定向到`https://www.example.com`，然后再继续访问网站。

这样的访问方式并不十分安全，因为在第一次HTTP请求和重定向的时候，攻击者可以以中间人的方式劫持这次请求，从而完成中间人攻击。

HSTS是一种web安全协议。采用该协议的网站将保证访问者发起的请求是HTTPS类型。
HSTS一般是由服务器发送给客户端，强制客户端（如浏览器）使用HTTPS与服务器创建连接。

```javascript {.line-numbers}
语法：
Strict-Transport-Security: max-age=<expire-time>
Strict-Transport-Security: max-age=<expire-time>; includeSubDomains
Strict-Transport-Security: max-age=<expire-time>; preload

max-age:设置在浏览器收到这个请求后的<expire-time>秒的时间内凡是访问这个域名下的请求都使用HTTPS请求。
includeSubDomains:如果这个可选的参数被指定，那么说明此规则也适用于该网站的所有子域名。
preload:查看 Preloading Strict Transport Security 获得详情。不是标准的一部分。
```
比如，https://xxx 的响应头含有Strict-Transport-Security: max-age=31536000; includeSubDomains。这意味着两点：
* 在接下来的一年（即31536000秒）中，浏览器只要向xxx或其子域名发送HTTP请求时，必须采用HTTPS来发起连接。比如，用户点击超链接或在地址栏输入 http://xxx/ ，浏览器应当自动将 http 转写成 https，然后直接向 https://xxx/ 发送请求。
* 在接下来的一年中，如果 xxx 服务器发送的TLS证书无效，用户不能忽略浏览器警告继续访问网站

服务器开启HSTS的方法是，当客户端通过HTTPS发出请求时，在服务器返回的超文本传输协议响应头中包含Strict-Transport-Security字段。非加密传输时设置的HSTS字段无效。

#### HSTS作用
HSTS可以用来抵御SSL剥离攻击。SSL剥离攻击是中间人攻击的一种，由Moxie Marlinspike于2009年发明。他在当年的黑帽大会上发表的题为“New Tricks For Defeating SSL In Practice”的演讲中将这种攻击方式公开。SSL剥离的实施方法是阻止浏览器与服务器创建HTTPS连接。它的前提是用户很少直接在地址栏输入https://，用户总是通过点击链接或3xx重定向，从HTTP页面进入HTTPS页面。所以攻击者可以在用户访问HTTP页面时替换所有https://开头的链接为http://，达到阻止HTTPS的目的。

HSTS可以很大程度上解决SSL剥离攻击，因为只要浏览器曾经与服务器创建过一次安全连接，之后浏览器会强制使用HTTPS，即使链接被换成了HTTP。

另外，如果中间人使用自己的自签名证书来进行攻击，浏览器会给出警告，但是许多用户会忽略警告。HSTS解决了这一问题，一旦服务器发送了HSTS字段，用户将不再允许忽略警告。

#### HSTS不足
用户首次访问某网站是不受HSTS保护的。这是因为首次访问时，浏览器还未收到HSTS，所以仍有可能通过明文HTTP来访问。解决这个不足目前有两种方案，一是浏览器预置HSTS域名列表，Google Chrome、Firefox、Internet Explorer和Spartan实现了这一方案。二是将HSTS信息加入到域名系统记录中。但这需要保证DNS的安全性，也就是需要部署域名系统安全扩展。截至2014年这一方案没有大规模部署。

由于HSTS会在一定时间后失效（有效期由max-age指定），所以浏览器是否强制HSTS策略取决于当前系统时间。部分操作系统经常通过网络时间协议更新系统时间，如Ubuntu每次连接网络时，OS X Lion每隔9分钟会自动连接时间服务器。攻击者可以通过伪造NTP信息，设置错误时间来绕过HSTS。解决方法是认证NTP信息，或者禁止NTP大幅度增减时间。比如Windows 8每7天更新一次时间，并且要求每次NTP设置的时间与当前时间不得超过15小时。

例子：
清空浏览器缓存，然后输入`tmall.com`，发现有三次访问记录，如下图：
```
```

### 参考链接
https://www.fxw.la/news/8137.html
https://blog.risingstack.com/node-js-security-checklist/