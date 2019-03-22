
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [HTTP -> Response Header -> X-XSS-Protection](#http-response-header-x-xss-protection)
	* [1. X-XSS-Protection语法](#1-x-xss-protection语法)
	* [2. X-XSS-Protection使用方式](#2-x-xss-protection使用方式)
	* [3. X-XSS-Protection产生背景](#3-x-xss-protection产生背景)
	* [4. X-XSS-Protection作用原理](#4-x-xss-protection作用原理)
	* [5. X-XSS-Protection不足](#5-x-xss-protection不足)
	* [6. 总结](#6-总结)
	* [参考链接](#参考链接)

<!-- /code_chunk_output -->

# HTTP -> Response Header -> X-XSS-Protection
X-XSS-Protection是HTTP响应头中的一个字段，该字段的主要功能是通知浏览器当发现跨站脚本时要做哪种操作。

## 1. X-XSS-Protection语法
* X-XSS-Protection: 0
* X-XSS-Protection: 1
* X-XSS-Protection: 1; mode=block

**0**: 禁止XSS过滤。
**1**: 启用XSS过滤，如果检测到跨站脚本攻击，浏览器将删除不安全的内容。
**1;mode=block**: 启用XSS过滤。如果检测到跨站脚本攻击，浏览器不会删除不安全的内容，而是阻止页面加载。

如果用户没有指定浏览器行为，浏览器默认使用 X-XSS-Protection: 1 模式。

## 2. X-XSS-Protection使用方式
在服务器的响应头中添加该字段并设置对应的值。
```javascript {.line-numbers}
// 伪代码
response.header.append({
    key: "X-XSS-Protection", 
    value: "X-XSS-Protection: 1; mode=block"
})
```

## 3. X-XSS-Protection产生背景
对客户端输入内容的不信任，导致X-XSS-Protection出现。

## 4. X-XSS-Protection作用原理
浏览器通过设置X-XSS-Protection，可以删除不受信任的代码，或者阻止有问题的页面加载，从而避免XSS攻击。

## 5. X-XSS-Protection不足
如果用户没有设置过X-XSS-Protection的值，浏览器会使用默认值 `1` 代替，使用默认值时，会一些问题出现：
* 扩大攻击面积：默认设置扩大了攻击面，比如攻击者可以利用这个默认设置选择性的删除页面中某些脚本。
* 引入新的安全漏洞：攻击者可以修改页面文档标签，破坏原始文档结构，导致浏览器误删某些元素，达到攻击的效果。
* 仍就会被绕过：即便设置了xss防御，也仍就有办法可以绕过xss防御。

## 6. 总结
X-XSS-Protection要根据具体项目具体设置，不恰当的引用某些js，可能会导致X-XSS-Protection误伤。

## 参考链接
1. https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/X-XSS-Protection