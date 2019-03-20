
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [HTTP X-Frame-Options](#http-x-frame-options)
	* [1. 为什么要有X-Frame-Options](#1-为什么要有x-frame-options)
	* [2. X-Frame-Options](#2-x-frame-options)
	* [3. X-Frame-Options总结](#3-x-frame-options总结)

<!-- /code_chunk_output -->



# HTTP X-Frame-Options
## 1. 为什么要有X-Frame-Options
X-Frame-Options主要是用来解决点击劫持(ClickJacking)相关的安全问题。

**点击劫持(也被称为UI覆盖)原理**：利用iframe标签的透明属性，使非法网站的某些操作与正常网站的操作按钮重合，当用户点击该按钮时，用户以为自己点击的是正常的按钮，实际上点击的是非法网站某个按钮。

**点击劫持防御**：通过设置HTTP响应头中的X-Frame-Options字段来组织其他网站引用本网站。

## 2. X-Frame-Options
X-Frame-Options有三个值：
* DENY:表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许。
* SAMEORIGIN:表示该页面可以在相同域名页面的 frame 中展示。
* ALLOW-FROM uri:表示该页面可以在指定来源的 frame 中展示。

## 3. X-Frame-Options总结
通过设置不同权限的X-Frame-Options值，以iframe形式引用会有不同的效果。
X-Frame-Options非官方标准，但已经被大多数浏览器采用并实施，是事实上的标准。