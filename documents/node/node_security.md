
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [Configuration Management](#configuration-management)
	* [一、Security HTTP Headers](#一-security-http-headers)

<!-- /code_chunk_output -->

# Configuration Management

## 一、Security HTTP Headers
下列这些和安全相关的HTTP Headers需要设置：
* **Strict-Transport-Security**: 强制客户端（如浏览器）使用HTTPS与服务器创建连接。参考链接[Strict-Transport-Security记录](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/http_response_header_Strick-Transport-Security.md)。
* **X-Frame-Options**: 提供点击劫持保护。参考链接[X-Frame-Options记录](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/http_response_header_X-Frame-Options.md)。
* **X-XSS-Protection**: 启用web浏览器中内置的跨站脚本筛选器。参考链接[X-XSS-Protection记录](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/http_response_header_X-XSS-Protection.md)。
* **X-Content-Type-Options**: 禁止浏览器中针对MIME与Content-Type类型不符的嗅探攻击。参考链接[X-Content-Type-Options](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/http_response_header_X-Content-Type-Options.md)。