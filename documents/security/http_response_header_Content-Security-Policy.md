
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [HTTP -> Response Header -> Content-Security-Policy](#http-response-header-content-security-policy)
	* [1. Content-Security-Policy语法](#1-content-security-policy语法)
		* [1.1 `<policy-directive>`指令名](#11-policy-directive指令名)
			* [1.1.1 获取型指令](#111-获取型指令)
				* [1.1.1.1 child-src](#1111-child-src)
				* [1.1.1.2 connect-src](#1112-connect-src)
				* [1.1.1.3 default-src](#1113-default-src)
				* [1.1.1.4 font-src](#1114-font-src)
				* [1.1.1.5 frame-src](#1115-frame-src)
				* [1.1.1.6 img-src](#1116-img-src)
				* [1.1.1.7 manifest-src](#1117-manifest-src)
				* [1.1.1.8 media-src](#1118-media-src)
				* [1.1.1.9 object-src](#1119-object-src)
				* [1.1.1.10 script-src](#11110-script-src)
				* [1.1.1.11 style-src](#11111-style-src)
				* [1.1.1.12 worker-src](#11112-worker-src)
			* [1.1.2 文档型指令](#112-文档型指令)
				* [1.1.2.1 base-uri](#1121-base-uri)
				* [1.1.2.2 plugin-types](#1122-plugin-types)
				* [1.1.2.3 sandbox](#1123-sandbox)
				* [1.1.2.4 disown-opener](#1124-disown-opener)
			* [1.1.3 导航型指令](#113-导航型指令)
				* [1.1.3.1 form-action](#1131-form-action)
				* [1.1.3.2 frame-ancestors](#1132-frame-ancestors)
				* [1.1.3.3 navigation-to](#1133-navigation-to)
			* [1.1.4 报告型指令](#114-报告型指令)
				* [1.1.4.1 report-uri](#1141-report-uri)
				* [1.1.4.2 report-to](#1142-report-to)
			* [1.1.5 其他指令](#115-其他指令)
				* [1.1.5.1 block-all-mixed-content](#1151-block-all-mixed-content)
				* [1.1.5.2 require-sri-for](#1152-require-sri-for)
				* [1.1.5.3 upgrade-insecure-requests](#1153-upgrade-insecure-requests)
		* [1.2 `<source>`源](#12-source源)
			* [1.2.1 `<host-source>`](#121-host-source)
			* [1.2.2 `<scheme-source>`](#122-scheme-source)
			* [1.2.3 `'self'`](#123-self)
			* [1.2.4 'unsafe-inline'](#124-unsafe-inline)
			* [1.2.5 `'unsafe-eval'](#125-unsafe-eval)
			* [1.2.6 'none'](#126-none)
			* [1.2.7 'nonce-<base64值>'](#127-nonce-base64值)
			* [1.2.8 `<hash-source>`](#128-hash-source)
			* [1.2.9 'strict-dynamic'](#129-strict-dynamic)
	* [2. Content-Security-Policy使用方式](#2-content-security-policy使用方式)
	* [3. Content-Security-Policy产生背景](#3-content-security-policy产生背景)
	* [4. Content-Security-Policy作用原理](#4-content-security-policy作用原理)
		* [4.1 解决XSS攻击](#41-解决xss攻击)
		* [4.2 数据包嗅探攻击](#42-数据包嗅探攻击)
		* [4.3 CSP默认特性](#43-csp默认特性)
			* [4.3.1 阻止内联代码执行](#431-阻止内联代码执行)
			* [4.3.2 EVAL相关功能被禁用](#432-eval相关功能被禁用)
	* [5. 常见例子](#5-常见例子)
	* [参考链接](#参考链接)

<!-- /code_chunk_output -->
# HTTP -> Response Header -> Content-Security-Policy
Content-Security-Policy是一个HTTP响应头，该响应头的主要功能是允许站点管理者在指定的页面上控制用户代理(带有src属性的标签，例如img、js、css等)的访问权限，毕竟这些标签是不受同源策略的限制，从而达到预防跨站脚本(XSS)和数据注入攻击效果。

其主要有两点：
* 通过白名单方式告诉浏览器允许/不允许加载哪些资源。
* 向服务器举报不安全的资源。

## 1. Content-Security-Policy语法
Content-Security-Policy: `<policy-directive>` `<source>`; 

### 1.1 `<policy-directive>`指令名
指令共分为五中类型：
* 获取型指令 - Fetch Directives
* 文档型指令 - Document Directives
* 导航型指令 - Navigation Directives
* 报告型指令 - Report Directives
* 其他指令   - Other Directives
#### 1.1.1 获取型指令
##### 1.1.1.1 child-src
为 web workers和其他内嵌浏览器内容定义 合法的源，例如用`<frame>`和`<iframe>`加载到页面的内容。
给定如下CSP头部:
```
Content-Security-Policy: child-src https://example.com/
```

如下的连接请求会被阻塞且不会加载。
```javascript
<iframe src="https://not-example.com"></iframe>

<script>
  var blockedWorker = new Worker("data:application/javascript,...");
</script>
```

##### 1.1.1.2 connect-src
用于控制允许通过脚本接口加载的链接地址。其中受到影响的API如下: 
* `<a>` ping,
* Fetch,
* XMLHttpRequest,
* WebSocket
* EventSource.

给定如下CSP头部:
```
Content-Security-Policy: connect-src https://example.com/
```

如下的连接请求会被阻塞且不会加载。
```javascript
<a ping="https://not-example.com">

<script>
  var xhr = new XMLHttpRequest(); 
  xhr.open('GET', 'https://not-example.com/'); 
  xhr.send();

  var ws = new WebSocket("https://not-example.com/");

  var es = new EventSource("https://not-example.com/"); 

  navigator.sendBeacon("https://not-example.com/", { ... });
</script>
```

##### 1.1.1.3 default-src
为其他 CSP 拉取指令（fetch directives）提供备选项。对于以下列出的指令，假如不存在的话，那么用户代理会查找并应用 default-src 指令的值。
* child-src
* connect-src
* font-src
* frame-src
* img-src
* manifest-src
* media-src
* object-src
* script-src
* style-src
* worker-src

给定如下CSP头部:
```
Content-Security-Policy: default-src 'self'; script-src https://example.com
```

如下的连接请求会被阻塞且不会加载。
```
Content-Security-Policy: connect-src 'self'; 
                         font-src 'self'; 
                         frame-src 'self'; 
                         img-src 'self'; 
                         manifest-src 'self'; 
                         media-src 'self'; 
                         object-src 'self'; 
                         script-src https://example.com; 
                         style-src 'self'; 
                         worker-src 'self'
```

##### 1.1.1.4 font-src
限制通过@font-face加载的字体源。
给定如下CSP头部:
```
Content-Security-Policy: font-src https://example.com/
```

如下的连接请求会被阻塞且不会加载。
```javascript
<style>
  @font-face { 
    font-family: "MyFont"; 
    src: url("https://not-example.com/font"); 
  } 
  body { 
    font-family: "MyFont"; 
  } 
</style>
```

##### 1.1.1.5 frame-src
限制通过类似`<frame>`和`<iframe>`标签加载的内嵌内容源。
给定如下CSP头部:
```
Content-Security-Policy: frame-src https://example.com/
```

如下的连接请求会被阻塞且不会加载。
```javascript
<iframe src="https://not-example.com/"></iframe>
```

##### 1.1.1.6 img-src
限制图片和图标源。
给定如下CSP头部:
```
Content-Security-Policy: img-src https://example.com/
```

如下的连接请求会被阻塞且不会加载。
```javascript
<img src="https://not-example.com/foo.jpg" alt="example picture">
```

##### 1.1.1.7 manifest-src
限制 application manifest 文件源。
给定如下CSP头部:
```
Content-Security-Policy: manifest-src https://example.com/
```

如下的连接请求会被阻塞且不会加载。
```javascript
<link rel="manifest" href="https://not-example.com/manifest">
```

##### 1.1.1.8 media-src
限制通过`<audio>`或`<video>`标签加载的媒体文件源。
给定如下CSP头部:
```
Content-Security-Policy: media-src https://example.com/
```

如下的连接请求会被阻塞且不会加载。
```javascript
<audio src="https://not-example.com/audio"></audio>

<video src="https://not-example.com/video"> 
  <track kind="subtitles" src="https://not-example.com/subtitles"> 
</video>
```

##### 1.1.1.9 object-src
限制通过`<object>`, `<embed>`，`<applet>`标签加载源。
给定如下CSP头部:
```
Content-Security-Policy: object-src https://example.com/
```

如下的连接请求会被阻塞且不会加载。
```javascript
<embed src="https://not-example.com/flash"></embed>
<object data="https://not-example.com/plugin"></object> 
<applet archive="https://not-example.com/java"></applet>
```

##### 1.1.1.10 script-src
限制javascript源。
**Violation case**:
给定如下CSP头部:
```
Content-Security-Policy: script-src https://example.com/
```

如下的连接请求会被阻塞且不会加载。
```javascript
<script src="https://not-example.com/js/library.js"></script>
```

**Unsafe inline script**:
注意，inline的事件处理也会被阻塞，你应该使用addEventListener代替onclick事件：
```javascript
// 下面的事件会被阻塞
<button id="btn" onclick="doSomething()">
// 使用如下方法代替
document.getElementById("btn").addEventListener('click', doSomething);
```
如果想支持inline sciprt和inline event hander，请使用'unsafe-inline'，nonce-source 或 hash-source。
```javascript
// 如果设置如下头
Content-Security-Policy: script-src 'unsafe-inline';
// 则如下语句可以生效
<script> 
  var inline = 1; 
</script>

// 如果设置如下头
Content-Security-Policy: script-src 'nonce-2726c7f26c'
// 则如下语句可以生效
<script nonce="2726c7f26c">
  var inline = 1;
</script>

// 如果设置如下头
Content-Security-Policy: script-src 'sha256-B2yPHKaXnvFWtRChIbabYmUBFZdVfKKXHbWtWidDVF8='
// 则如下语句可以生效
<script>var inline = 1;</script>
```

**Unsafe eval expressions**:
如果 script-src 'unsafe-eval' 没有指定，那么如下的请求会被阻塞。
* eval()
* Function()
* When passing a string literal like to methods like: window.setTimeout("alert(\"Hello World!\");", 500);
    * window.setTimeout
    * window.setInterval
    * window.setImmediate
* window.execScript  (IE < 11 only)

##### 1.1.1.11 style-src
限制层叠样式表文件源。
**Violation cases**：
给定如下CSP头部:
```
Content-Security-Policy: style-src https://example.com/
```

如下的连接请求会被阻塞且不会加载。
```javascript
<link href="https://not-example.com/styles/main.css" rel="stylesheet" type="text/css" />

<style>
#inline-style { background: red; }
</style>

<style>
  @import url("https://not-example.com/styles/print.css") print;
</style>

// using the Link header:
Link: <https://not-example.com/styles/stylesheet.css>;rel=stylesheet

<div style="display:none">Foo</div>

document.querySelector('div').setAttribute('style', 'display:none;');
document.querySelector('div').style.cssText = 'display:none;';

document.querySelector('div').style.display = 'none';
```

**Unsafe inline styles**：
```javascript
// 如果设置如下头
Content-Security-Policy: style-src 'unsafe-inline';
// 则如下语句可以生效
<style>
#inline-style { background: red; }
</style>

<div style="display:none">Foo</div>

// 如果设置如下头
Content-Security-Policy: style-src 'nonce-2726c7f26c'
// 则如下语句可以生效
<style nonce="2726c7f26c">
#inline-style { background: red; }
</style>

// 如果设置如下头
Content-Security-Policy: style-src 'sha256-a330698cbe9dc4ef1fb12e2ee9fc06d5d14300262fa4dc5878103ab7347e158f'
// 则如下语句可以生效
<style>#inline-style { background: red; }</style>
```

**Unsafe style expressions**：
如果style-src 'unsafe-eval'没有指定, 下列方法会被阻塞:
```javascript
CSSStyleSheet.insertRule()
CSSGroupingRule.insertRule()
CSSStyleDeclaration.cssText
```


##### 1.1.1.12 worker-src
限制Worker, SharedWorker, 或者 ServiceWorker脚本源。
给定如下CSP头部:
```
Content-Security-Policy: worker-src https://example.com/
```

如下的连接请求会被阻塞且不会加载。
```javascript
<script> 
  var blockedWorker = new Worker("data:application/javascript,..."); 
  blockedWorker = new SharedWorker("https://not-example.com/"); 
  navigator.serviceWorker.register('https://not-example.com/sw.js'); 
</script>

```

#### 1.1.2 文档型指令
##### 1.1.2.1 base-uri
该指令限制了可以应用于一个文档的 `<base>` 元素的 URL。假如指令值为空，那么任何 URL 都是允许的。如果指令不存在，那么用户代理会使用 `<base>` 元素中的值。 
如果源网站地址是http://example.com/，base中使用其他网站会被拒绝。
```javascript
// Origin http://example.com/
<meta http-equiv="Content-Security-Policy" content="base-uri 'self'">
<base href="http://not-example.com/">
```

##### 1.1.2.2 plugin-types
限制一系列可以通过一些限制类型的资源加载内嵌到DOM的插件。
##### 1.1.2.3 sandbox
允许类似{HTMLElement("iframe")}} sandbox sandbox 属性，
##### 1.1.2.4 disown-opener 
确保资源在操作的时候能够脱离父页面。

#### 1.1.3 导航型指令
##### 1.1.3.1 form-action
限制能被用来作为给定上下文的表单提交的目标 URL （就是限制 form 的 action 属性的链接地址）
```javascript
<meta http-equiv="Content-Security-Policy" content="form-action 'none'">

<form action="javascript:alert('Foo')" id="form1" method="post"> 
  <input type="text" name="fieldName" value="fieldValue"> 
  <input type="submit" id="submit" value="submit"> 
</form>

// Error: Refused to send form data because it violates the following 
// Content Security Policy directive: "form-action 'none'".
```
##### 1.1.3.2 frame-ancestors
指定可能嵌入页面的有效父项`<frame>, <iframe>, <object>, <embed>, <applet>`.
当该指令设置为'none'时，其作用类似于X-Frame-Options: DENY 

##### 1.1.3.3 navigation-to 
限制文档可以通过以下任何方式访问URL (a, form, window.location, window.open, etc.)

#### 1.1.4 报告型指令
##### 1.1.4.1 report-uri 
~~当出现可能违反CSP的操作时，让客户端提交报告。这些违规报告会以JSON文件的格式通过POST请求发送到指定的URI。~~ Deprecated

##### 1.1.4.2 report-to 
Fires a SecurityPolicyViolationEvent.

#### 1.1.5 其他指令
##### 1.1.5.1 block-all-mixed-content
在当前页面为通过 HTTPS 协议加载的情况下禁止通过 HTTP 渠道加载任何资源。

任何混合类型的资源请求都是被禁止的，包括混合活动内容和混合被动内容。这一条也适用于 `<iframe>` 中的文档，确保整体页面都不包含混合内容。

upgrade-insecure-requests 指令会在 block-all-mixed-content 之前执行；如果前者执行成功，后者就不再发挥任何作用。推荐的做法是设置二者之一，而不是全部。

##### 1.1.5.2 require-sri-for 
指示客户端在页面上对脚本或样式使用子资源完整性策略。
如果你通过如下指令将站点设置为要求脚本和资源满足SRI策略: 
```
Content-Security-Policy: require-sri-for script style
```
```javascript
// 下面的脚本会被加载，因为它拥有有效的完整性属性。
<script src="https://code.jquery.com/jquery-3.1.1.slim.js"
        integrity="sha256-5i/mQ300M779N2OVDrl16lbohwXNUdzL/R2aVUXyXWA="
        crossorigin="anonymous">
</script>

// 下面的脚本不会被加载，因为没有完整性属性。
<script src="https://code.jquery.com/jquery-3.1.1.slim.js"></script>
```
##### 1.1.5.3 upgrade-insecure-requests
指示客户端将该站点的所有不安全URL（通过HTTP提供的URL）视为已被替换为安全URL（通过HTTPS提供的URL）。该指令适用于需要重写大量不安全的旧版URL的网站。

upgrade-insecure-requests指令在 block-all-mixed-content 之前被执行，如果其被设置，后者实际上是空操作。可以设置其中一个，但不能同时设置。

upgrade-insecure-requests指令不能保证用户浏览你的网站时用的是HTTPS，所以它不能代替HTTP Strict-Transport-Security。

```javascript
// 一旦将下述头部设置在计划从HTTP迁移到HTTPS的example.com域名上, 非跳转(non-navigational)的不安全资源请求会自动升级到H(non-navigational)的不安全资源请求会自动升级到HTTPS（包括第当前域名以及第三方请求）。
// header
Content-Security-Policy: upgrade-insecure-requests;

// meta tag
<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">

// 这些URL在请求发送之前都会被改写成HTTPS，也就意味着不安全的请求都不会发送出去。注意，如果请求的资源在HTTPS情况下不可用，则该请求将失败, 其也不能回退到HTTP。
<img src="http://example.com/image.png">
<img src="http://not-example.com/image.png">
```

通过 Content-Security-Policy-Report-Only  HTTP头部和 report-uri 指令，您可以设置执行策略和报告策略，如下所示：
```
Content-Security-Policy: upgrade-insecure-requests; default-src https: 
Content-Security-Policy-Report-Only: default-src https:; report-uri /endpoint
```

### 1.2 `<source>`源
源可以指定多个，必须是以下内容之一：
#### 1.2.1 `<host-source>`
以域名或者 IP 地址表示的主机名，外加可选的 URL 协议名（URL scheme）以及端口号。站点地址中可能会包含一个可选的前置通配符（星号 * ），同时也可以将通配符应用于端口号，表示在这个源中可以使用任意合法的端口号。举例说明：
* `http://*.example.com` - 匹配使用http协议的、从`example.com`的任意子域加载的资源。
* `mail.example.com:443` - 匹配对`mail.example.com`上的443端口的资源访问。
* `https://store.example.com` - 匹配对使用了https协议的`store.example.com`的访问。
#### 1.2.2 `<scheme-source>`
协议名如 'http:' 或者 'https:'。必须带有冒号，不要有单引号。同时你还可以指定数据协议（data schema）（不推荐使用）。
* data: 允许 data: URIs 作为内容的源。这是不安全的。攻击者可以注入任意 data: URI 。不要轻易使用这种形式的源，尤其是脚本，绝对不要使用。
* mediastream: 允许 mediastream: URIs 作为内容的源。
* blob: 允许 blob: URIs 作为内容的源。
* filesystem: 允许 filesystem: URIs 作为内容的源。
#### 1.2.3 `'self'`
指向与要保护的文件所在的源，包括相同的 URL scheme 与端口号。必须有单引号。一些浏览器会特意排除 blob 与 filesystem 。需要设定这两种内容类型的站点可以在 Data 属性中进行设定。
#### 1.2.4 'unsafe-inline'
允许使用内联资源，例如内联 `<script>`  元素（javascript: URL）、内联事件处理器以及内联 `<style>` 元素。必须有单引号。
#### 1.2.5 `'unsafe-eval'
允许使用 eval() 以及相似的函数来从字符串创建代码。必须有单引号。
#### 1.2.6 'none'
不允许任何内容。 必须有单引号。
#### 1.2.7 'nonce-<base64值>'
特定使用一次性加密内联脚本的白名单。服务器必须在每一次传输政策时生成唯一的一次性值。否则将存在绕过资源政策的可能。请参见不安全的内联脚本查看示例。
#### 1.2.8 `<hash-source>`
使用 sha256、sha384 或 sha512 编码过的内联脚本或样式。其由用短划线分隔的两部分组成: 用于创建哈希的加密算法, 以及脚本或样式base64编码的哈希值。当生成哈希值的时候，不要包含 `<script>` 或 `<style>` 标签，同时注意字母大小写与空格——包括首尾空格——都是会影响生成的结果的。请参见不安全的内联脚本。
#### 1.2.9 'strict-dynamic'
strict-dynamic 指定对于含有标记脚本(通过附加一个随机数或散列)的信任，应该传播到由该脚本加载的所有脚本。与此同时，任何白名单以及源表达式例如 'self'  或者  'unsafe-inline' 都会被忽略。

## 2. Content-Security-Policy使用方式
CSP可以由两种方式指定：HTTP Header和HTML，如果HTTP头与meta标签同时定义了CSP，则会优先采用HTTP头的。

1. 在服务器的响应头中添加该字段并设置对应的值：
```javascript {.line-numbers}
// 伪代码
response.header.append({
    key: "Content-Security-Policy", 
    value: "default-src: 'self' https://google.com; child-src 'none'; object-src 'none';"
})
```

2. 通过定义在 HTML meta标签中使用：
```javascript
<meta http-equiv="content-security-policy" content="default-src: 'self' https://google.com; child-src 'none'; object-src 'none'">
```

## 3. Content-Security-Policy产生背景
浏览器为了避免XSS攻击使用了同源策略，但是一些带有src的标签不受同源策略的影响，可以引用外部资源，这就给了攻击者可乘之机，在某些时候修改带有src标签的指向，加载了非法代码。

在这种情况下，开发者提出了这样的需求，要求可以指定这些能改变资源链接的标签的白名单。

## 4. Content-Security-Policy作用原理
### 4.1 解决XSS攻击
CSP的主要目标是减少和报告XSS攻击 ，XSS攻击利用了浏览器对于从服务器所获取的内容的信任。恶意脚本在受害者的浏览器中得以运行，因为浏览器信任其内容来源，即使有的时候这些脚本并非来自于它本该来的地方。

CSP通过指定有效域——即浏览器认可的可执行脚本的有效来源——使服务器管理者有能力减少或消除XSS攻击所依赖的载体。一个CSP兼容的浏览器将会仅执行从白名单域获取到的脚本文件，忽略所有的其他脚本 (包括内联脚本和HTML的事件处理属性)。

作为一种终极防护形式，始终不允许执行脚本的站点可以选择全面禁止脚本执行。

### 4.2 数据包嗅探攻击
除限制可以加载内容的域，服务器还可指明哪种协议允许使用；比如服务器可指定所有内容必须通过HTTPS加载。一个完整的数据安全传输策略不仅强制使用HTTPS进行数据传输，也为所有的cookie标记安全标识 cookies with the secure flag，并且提供自动的重定向使得HTTP页面导向HTTPS版本。网站也可以使用  Strict-Transport-Security  HTTP头部确保连接它的浏览器只使用加密通道。

### 4.3 CSP默认特性
#### 4.3.1 阻止内联代码执行
CSP除了使用白名单机制外，默认配置下阻止内联代码执行是防止内容注入的最大安全保障。
```javascript
// 这里的内联代码包括：<script>块内容，内联事件，内联样式。
// 1. 对于<script>块内容是完全不能执行的。例如：
<script>getyourcookie()</script>

// 2. 内联事件。
<a href="" onclick="handleClick();"></a> 
<a href="javascript:handleClick();"></a>

// 3.内联样式
<div style="display:none"></div>
```
虽然CSP中已经对script-src和style-src提供了使用”unsafe-inline”指令来开启执行内联代码，但为了安全起见还是慎用”unsafe-inline”。

#### 4.3.2 EVAL相关功能被禁用
用户输入字符串，然后经过eval()等函数转义进而被当作脚本去执行。这样的攻击方式比较常见。于是乎CSP默认配置下，eval() , newFunction(), setTimeout([string], …) 和setInterval([string], …)都被禁止运行。
比如：
```javascript
alert(eval("foo.bar.baz"));
window.setTimeout("alert(‘hi‘)", 10);
window.setInterval("alert(‘hi‘)", 10); 
new Function("return foo.bar.baz");
```
如果想执行可以把字符串转换为内联函数去执行。
```javascript
alert(foo && foo.bar && foo.bar.baz);
window.setTimeout(function() { alert(‘hi‘); }, 10);
window.setInterval(function() { alert(‘hi‘); }, 10);
function() { return foo && foo.bar && foo.bar.baz };
```
同样CSP也提供了”unsafe-eval”去开启执行eval()等函数，但强烈不建议去使用”unsafe-eval”这个指令。

## 5. 常见例子
**例 1**
一个网站管理者想要所有内容均来自站点的同一个源 (不包括其子域名):
```
Content-Security-Policy: default-src 'self'
```

**例 2**
一个网站管理者允许内容来自信任的域名及其子域名 (域名不必须与CSP设置所在的域名相同)
```
Content-Security-Policy: default-src 'self' *.trusted.com
```

**例 3**
一个网站管理者允许网页应用的用户在他们自己的内容中包含来自任何源的图片, 但是限制音频或视频需从信任的资源提供者(获得)，所有脚本必须从特定主机服务器获取可信的代码.
```
Content-Security-Policy: default-src 'self'; image-src *; media-src media1.com media2.com; script-src target.com;
```
在这里，各种内容默认仅允许从文档所在的源获取, 但存在如下例外:
* 图片可以从任何地方加载(注意 "*" 通配符)。
* 多媒体文件仅允许从 `media1.com` 和 `media2.com` 加载(不允许从这些站点的子域名)。
* 可运行脚本仅允许来自于`target.com`。

**例 4**
一个线上银行网站的管理者想要确保网站的所有内容都要通过SSL方式获取，以避免攻击者窃听用户发出的请求。
```javascript
// 该服务器仅允许通过HTTPS方式并仅从onlinebanking.jumbobank.com域名来访问文档。
Content-Security-Policy: default-src https://onlinebanking.jumbobank.com
```

**例 5**
 一个在线邮箱的管理者想要允许在邮件里包含HTML，同样图片允许从任何地方加载，但不允许JavaScript或者其他潜在的危险内容(从任意位置加载)。
```
Content-Security-Policy: default-src 'self' *.mailsite.com; img-src *
```
注意这个示例并未指定script-src。在此CSP示例中，站点通过 default-src 指令的对其进行配置，这也同样意味着脚本文件仅允许从原始服务器获取。

**例 6**
为降低部署成本，CSP可以部署为报告(report-only)模式。在此模式下，CSP策略不是强制性的，但是任何违规行为将会报告给一个指定的URI地址。此外，一个报告模式的头部可以用来测试一个修订后的未来将应用的策略而不用实际部署它。
你可以用Content-Security-Policy-Report-Only HTTP 头部来指定你的策略，像这样：
```
Content-Security-Policy-Report-Only: policy
```
如果Content-Security-Policy-Report-Only 头部和 Content-Security-Policy 同时出现在一个响应中，两个策略均有效。在Content-Security-Policy 头部中指定的策略有强制性 ，而Content-Security-Policy-Report-Only中的策略仅产生报告而不具有强制性。

支持CSP的浏览器将始终对于每个企图违反你所建立的策略都发送违规报告，如果策略里包含一个有效的report-uri 指令。
```
Content-Security-Policy: default-src 'self'; report-uri http://reportcollector.example.com/collector
```

作为报告的JSON对象报告包含了以下数据：
```
{
  "csp-report": {
    "document-uri": "http://example.com/signup.html",
    "referrer": "",
    "blocked-uri": "http://example.com/css/style.css",
    "violated-directive": "style-src cdn.example.com",
    "original-policy": "default-src 'none'; style-src cdn.example.com; report-uri /_/csp-reports"
  }
}
```
* **document-uri**：发生违规的文档的URI。
* **referrer**：违规发生处的文档引用（地址）。
* **blocked-uri**：被CSP阻止的资源URI。如果被阻止的URI来自不同的源而非文档URI，那么被阻止的资源URI会被删减，仅保留协议，主机和端口号。
* **violated-directive**：违反的策略名称。
* **original-policy**：在 Content-Security-Policy HTTP 头部中指明的原始策略。

## 参考链接
1. https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy__by_cnvoid
2. https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP
3. https://www.jianshu.com/p/a8b769e7d4bd
4. http://www.mamicode.com/info-detail-2440203.html