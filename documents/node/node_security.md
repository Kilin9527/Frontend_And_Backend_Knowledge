
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [Configuration Management](#configuration-management)
	* [一、Security HTTP Headers](#一-security-http-headers)

<!-- /code_chunk_output -->

# Configuration Management
## 一、Security HTTP Headers
下列这些和安全相关的HTTP Headers需要设置：
* **Strict-Transport-Security**: 强制客户端（如浏览器）使用HTTPS与服务器创建连接。参考链接[HSTS记录](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/http/HTTP_HEADERS_HSTS.md)。
* **X-Frame-Options**:
