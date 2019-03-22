
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [HTTP - Response Header - X-Content-Type-Options](#http-response-header-x-content-type-options)
	* [1. X-Content-Type-Options语法](#1-x-content-type-options语法)
	* [2. X-Content-Type-Options使用方式](#2-x-content-type-options使用方式)
	* [3. X-Content-Type-Options产生背景](#3-x-content-type-options产生背景)
	* [4. X-Content-Type-Options作用原理](#4-x-content-type-options作用原理)
	* [5. 总结](#5-总结)
	* [参考链接](#参考链接)

<!-- /code_chunk_output -->

# HTTP - Response Header - X-Content-Type-Options
X-Content-Type-Options是HTTP响应头中的一个字段，该字段的主要功能是告诉浏览器一定要遵循在Content-Type首部中对MIME类型的设定，而不能对其进行修改。

## 1. X-Content-Type-Options语法
* X-Content-Type-Options: nosniff

**nosniff**：假如请求类型为以下两种，那么阻止请求的发生：
* 如果请求的是`style`资源，但是指定的MIME类型没有指明是`text/css`。
* 如果请求的是`script`资源，但是指定的MIME类型不属于js系的任一种。

## 2. X-Content-Type-Options使用方式
在服务器的响应头中添加该字段并设置对应的值。
```javascript {.line-numbers}
// 伪代码
response.header.append({
    key: "X-Content-Type-Options", 
    value: "nosniff"
})
```

## 3. X-Content-Type-Options产生背景
如果浏览器请求一个资源，例如png图片，但是响应头中的MIME类型没有设置类型，或者设置了一个错误的类型，如text/css，由于类型不匹配，此时浏览器会对该资源进行类型检查，例如读取前xx个字节的内容来判断是什么类型的文件(很多文件会将自己的类型信息写在文件最前面或者最后面的xx字节中)，这个过程叫做嗅探。

如果攻击者将图片的前xx字节内容变更，此时浏览器的读取过程就会十分危险，通过这种方式获取到用户信息的行为叫做嗅探攻击/混淆攻击。

所以，通过设置X-Content-Type-Options的nosniff属性，可以阻止对类型异常的文件的检查，直接拒绝其加载。

## 4. X-Content-Type-Options作用原理
* 如果stylesheet资源的MIME类型不属于下列之一，则阻止其加载：
    * text/css
* 如果javascript资源的MIME类型不属于下列之一，则阻止其加载：
    * application/ecmascript
    * application/javascript
    * application/x-javascript
    * text/ecmascript
    * text/javascript
    * text/jscript
    * text/x-javascript
    * text/vbs
    * text/vbscript

## 5. 总结
通过设置页面申请的资源的MIME类型，可以有效的避免针对浏览器的嗅探攻击。

X-Content-Type-Options只针对两种类型文件起作用，stylesheet和javascript。

## 参考链接
1. https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/X-Content-Type-Options