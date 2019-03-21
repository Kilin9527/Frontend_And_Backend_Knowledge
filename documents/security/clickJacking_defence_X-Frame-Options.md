
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [HTTP X-Frame-Options](#http-x-frame-options)
	* [1. 为什么要有X-Frame-Options](#1-为什么要有x-frame-options)
	* [2. X-Frame-Options](#2-x-frame-options)
	* [3. X-Frame-Options总结](#3-x-frame-options总结)

<!-- /code_chunk_output -->


# HTTP X-Frame-Options
## 1. 为什么要有X-Frame-Options
X-Frame-Options主要是用来解决[点击劫持](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/clickJacking_attack.md)相关的安全问题。

**点击劫持防御**：通过设置HTTP响应头中的X-Frame-Options字段来阻止其他网站引用本网站。

## 2. X-Frame-Options
X-Frame-Options有三个值：
* DENY:表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许。
* SAMEORIGIN:表示该页面可以在相同域名页面的 frame 中展示。
* ALLOW-FROM uri:表示该页面可以在指定来源的 frame 中展示。

## 3. X-Frame-Options总结
通过设置不同权限的X-Frame-Options值，以iframe形式引用会有不同的效果。
X-Frame-Options非官方标准，但已经被大多数浏览器采用并实施，是事实上的标准。