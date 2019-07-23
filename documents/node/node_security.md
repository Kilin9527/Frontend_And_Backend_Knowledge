
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [一、Configuration Management](#一-configuration-management)
	* [1. Security HTTP Headers](#1-security-http-headers)
	* [2. 客户端上的敏感数据](#2-客户端上的敏感数据)
* [二、Authentication](#二-authentication)
	* [1. 暴力破解保护(Brute Force Protection)](#1-暴力破解保护brute-force-protection)

<!-- /code_chunk_output -->

# 一、Configuration Management

## 1. Security HTTP Headers
下列这些和安全相关的HTTP Headers需要设置：
* **Strict-Transport-Security**: 强制客户端（如浏览器）使用HTTPS与服务器创建连接。参考链接[Strict-Transport-Security记录](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/http_response_header_Strick-Transport-Security.md)。
* **X-Frame-Options**: 提供点击劫持保护。参考链接[X-Frame-Options记录](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/http_response_header_X-Frame-Options.md)。
* **X-XSS-Protection**: 启用浏览器中内置的跨站脚本筛选器。参考链接[X-XSS-Protection记录](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/http_response_header_X-XSS-Protection.md)。
* **X-Content-Type-Options**: 禁止浏览器中针对MIME与Content-Type类型不符的嗅探攻击。参考链接[X-Content-Type-Options](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/http_response_header_X-Content-Type-Options.md)。
* **Content-Security-Policy**: 限制页面中带有src的标签的访问权限，限制内联样式和脚本等。参考链接[Content-Security-Policy](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/http_response_header_Content-Security-Policy.md)。

在Node中可以通过Helmet模块实现上述功能:
```javascript {.line-numbers}
var express = require('express');
var helmet = require('helmet');
 
var app = express();
app.use(helmet());
```

## 2. 客户端上的敏感数据
部署客户端程序时，请确保没有将API的秘钥和证书暴露在源代码当中，因为其他人可以读取到它。

这种问题并没有有效的自动化检查手动，但你可以通过以下两点来尝试：
* 使用Pull Request
* 定期的Code Review

# 二、Authentication
## 1. 暴力破解保护(Brute Force Protection)
暴力破解通过穷举法来不停尝试，直到满足条件为止，一个典型的例子是登录功能，暴力破解通过不停变换密码来登录同一个用户，直到登录成功。

在Node.js中，可以通过rate-limiting类的功能来约束用户在一定时间内的登录次数，通常对登录操作使用即可，当然也可以对所有的API进行约束。
