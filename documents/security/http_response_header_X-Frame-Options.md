
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [HTTP -> Response Header -> X-Frame-Options](#http-response-header-x-frame-options)
	* [1. X-Frame-Options语法](#1-x-frame-options语法)
	* [2. X-Frame-Options使用方式](#2-x-frame-options使用方式)
	* [3. X-Frame-Options产生背景](#3-x-frame-options产生背景)
	* [4. X-Frame-Options作用原理](#4-x-frame-options作用原理)
	* [5. 总结](#5-总结)
	* [参考链接](#参考链接)

<!-- /code_chunk_output -->

# HTTP -> Response Header -> X-Frame-Options
X-Frame-Options是一个HTTP响应头，该响应头的主要功能是告诉浏览器本网页是否可以在iframe中嵌套。

## 1. X-Frame-Options语法
* X-Frame-Options: DENY
* X-Frame-Options: SAMEORIGIN
* X-Frame-Options: ALLOW-FROM uri

**DENY**:表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许。
**SAMEORIGIN**:表示该页面可以在相同域名页面的 frame 中展示。
**ALLOW-FROM uri**:表示该页面可以在指定来源的 frame 中展示。

## 2. X-Frame-Options使用方式
在服务器的响应头中添加该字段并设置对应的值。
```javascript {.line-numbers}
// 伪代码
response.header.append({
    key: "X-Frame-Options", 
    value: "SAMEORIGIN"
})
```

## 3. X-Frame-Options产生背景
攻击者可以利用iframe/frame等标签嵌入非法网站，并通过设置透明度使用户看不到该非法网站，当用户在非法网站上操作时，可能会被窃取信息或跳转到其他网站，这种攻击方式叫做[点击劫持](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/attack_clickJacking.md)。

X-Frame-Options主要是用来解决点击劫持相关的安全问题。

## 4. X-Frame-Options作用原理
如果我有一个页面`http://mySite.com`的X-Frame-Options设置为`ALLOW-FROM http://mySiteOther.com`，那么只有`http://mySiteOther.com`中的iframe可以嵌套我的网页。其他任何网页嵌套我的网页都会被拒绝。

如果我有一个页面`http://mySite.com`的X-Frame-Options设置为`SAMEORIGIN`，那么只有与我的网页[同源](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/concepts/same_origin_policy.md)的页面才可以利用iframe嵌套我的网页。其他任何网页嵌套我的网页都会被拒绝。

如果我有一个页面`http://mySite.com`的X-Frame-Options设置为`DENY`，那么我的网页禁止在任何网中嵌套。

## 5. 总结
通过设置不同权限的X-Frame-Options值，以iframe形式引用会有不同的效果。

X-Frame-Options非官方标准，但已经被大多数浏览器采用并实施，是事实上的标准。

## 参考链接
1. https://developer.mozilla.org/zh-CN/docs/Web/HTTP/X-Frame-Options