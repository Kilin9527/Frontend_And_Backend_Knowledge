
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [Configuration Management](#configuration-management)
	* [一、Security HTTP Headers](#一-security-http-headers)
		* [1. Strict-Transport-Security](#1-strict-transport-security)
			* [HSTS概述](#hsts概述)
			* [HSTS作用](#hsts作用)
			* [HSTS不足](#hsts不足)

<!-- /code_chunk_output -->

# Configuration Management
## 一、Security HTTP Headers
下列这些和安全相关的HTTP Headers需要设置：
* **Strict-Transport-Security**: 强制客户端（如浏览器）使用HTTPS与服务器创建连接。
* **X-Frame-Options**:
