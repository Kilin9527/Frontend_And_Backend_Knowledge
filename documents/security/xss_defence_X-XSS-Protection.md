
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [XSS防御](#xss防御)
	* [XSS语法](#xss语法)
	* [XSS默认设置并不十分安全](#xss默认设置并不十分安全)

<!-- /code_chunk_output -->


# XSS防御
HTTP X-XSS-Protection 响应头是Internet Explorer，Chrome和Safari的一个功能，当检测到跨站脚本攻击 (XSS)时，浏览器将停止加载页面。

## XSS语法
X-XSS-Protection: 0
X-XSS-Protection: 1
X-XSS-Protection: 1; mode=block

**0**: 禁止XSS过滤。
**1**: 启用XSS过滤（通常浏览器是默认的）。如果检测到跨站脚本攻击，浏览器将清除页面不安全的部分（删除部分代码）。
**1;mode=block**: 启用XSS过滤。 如果检测到攻击，浏览器将不会清除页面，而是阻止页面加载。

## XSS默认设置并不十分安全
* 扩大攻击面积：默认设置扩大了攻击面，比如攻击者可以利用这个默认设置选择性的删除页面中某些脚本。
* 引入新的安全漏洞：攻击者可以将无害的标签变成有害的标签，因为过滤器有时候很傻，会不正确地替换了关键位置的字符，从而损害了原始文档的结构。通过精心制作的有效载荷，可以绕过属性下文的限制。
* 仍就会被绕过：即便设置了xss防御，也仍就有办法可以绕过xss防御。