
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [XSS(Cross site script)跨站脚本攻击学习记录](#xsscross-site-script跨站脚本攻击学习记录)
	* [1. XSS攻击概念](#1-xss攻击概念)
	* [2. XSS攻击发生的原因](#2-xss攻击发生的原因)
	* [3. XSS攻击分类](#3-xss攻击分类)
		* [3.1 反射型XSS攻击](#31-反射型xss攻击)
		* [3.2 存储型XSS攻击](#32-存储型xss攻击)
		* [3.3 基于DOM的XSS攻击](#33-基于dom的xss攻击)
	* [4. 攻击实例](#4-攻击实例)
	* [5. 防范措施](#5-防范措施)
	* [参考链接](#参考链接)

<!-- /code_chunk_output -->

# XSS(Cross site script)跨站脚本攻击学习记录

## 1. XSS攻击概念
XSS攻击是一种针web应用安全漏洞的攻击，恶意攻击者利用网站没有对用户提交数据进行转义处理或者过滤的缺点，进而添加一些代码，嵌入到web页面中去。其他用户访问会执行相应的嵌入代码，从而盗取用户资料、利用用户身份进行某种动作或者对访问者进行病毒侵害的一种攻击方式。

## 2. XSS攻击发生的原因
主要原因：过于相信客户端提交的信息。
解决办法：不要相信任何客户端提交的数据，对所有客户端提交的数据进行过滤处理。

## 3. XSS攻击分类
### 3.1 反射型XSS攻击
反射型XSS攻击，也叫非持久型XSS攻击，是指发生请求时，XSS攻击代码出现在请求URL中，用户点击攻击链接，服务器解析后响应，在返回的响应内容中出现攻击者的XSS代码，被浏览器执行。一来一去，XSS攻击脚本被web server反射回来给浏览器执行，所以称为反射型XSS。
* XSS攻击代码非持久性，也就是没有保存在web server中，而是出现在URL地址中。
* 非持久性，那么攻击方式就不同了。一般是攻击者通过邮件，聊天软件等等方式发送攻击URL，然后用户点击来达到攻击的。

### 3.2 存储型XSS攻击
存储型xss攻击，又称为持久型跨站点脚本，恶意代码存储在web server中，这样，每一个访问特定网页的用户，都会被攻击。

* XSS攻击代码存储于web server上。
* 攻击者，一般是通过网站的留言、评论、博客、日志等等功能(所有能够向web server输入内容的地方)，将攻击代码存储到web server上的。

### 3.3 基于DOM的XSS攻击
基于DOM的XSS，也就是web server不参与，仅仅涉及到浏览器的XSS。比如根据用户的输入来动态构造一个DOM节点，如果没有对用户的输入进行过滤，那么也就导致XSS攻击的产生。

## 4. 攻击实例
一个XSS攻击，留言类，简单注入javascript。

有个表单域：`<input type=“text” name=“content” value=“这里是用户填写的数据”>`

1、假若用户填写数据为：`<script>alert('foolish!')</script>`（或者`<script type="text/javascript" src="./xss.js"></script>`）

2、提交后将会弹出一个foolish警告窗口，接着将数据存入数据库。

3、等到别的客户端请求这个留言的时候，将数据取出显示留言时将执行攻击代码，将会显示一个foolish警告窗口。

## 5. 防范措施
参见xss防御一章。


## 参考链接
1. https://www.cnblogs.com/phpstudy2015-6/p/6767032.html
