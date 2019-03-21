
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [Click-Jacking 点击劫持](#click-jacking-点击劫持)
	* [1. 什么是点击劫持](#1-什么是点击劫持)
	* [2. 点击劫持过程](#2-点击劫持过程)
	* [3. 如何避免点击劫持事件](#3-如何避免点击劫持事件)

<!-- /code_chunk_output -->

# Click-Jacking 点击劫持
## 1. 什么是点击劫持
点击劫持也被称为UI覆盖攻击，是一种视觉上的欺骗手段，虽然受害者点击的是他所看到的网页，但其实他所点击的是被黑客精心构建的另一个置于原网页上面的透明页面，这种攻击利用了HTML中`<iframe>`标签的透明属性。

大概有两种方式：
* 一是攻击者使用一个透明的iframe，覆盖在一个网页上，然后诱使用户在该页面上进行操作，此时用户将在不知情的情况下点击透明的iframe页面。
* 二是攻击者使用一张图片覆盖在网页，遮挡网页原有位置的含义。

## 2. 点击劫持过程
1. 首先创建一个html页面，上面有一个美女图片和一个名为点击交友的按钮。如图：
![点击劫持_1](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/security/Click_Jacking_Attack_1.png?raw=true)

2. 如果用户点击该按钮，则发生点击劫持事件，实际上该按钮上面覆盖了一层透明的iframe图层，如图:
![点击劫持_2](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/security/Click_Jacking_Attack_2.png?raw=true)
例子中的点击劫持事件仅仅是让你在某一个贴吧中签到，但真实的点击劫持事件要比这个严重的多。

3. 看一下源代码：
```javascript {.line-numbers}
<!DOCTYPE html>
<head> </head>
<body>
  <img style="width:50%;height:auto;" src="" />
  <button style="width: 100px;height:50px;">点击交友</button>
  <iframe
    style="position: absolute;float: left;width: 1024px;height: 800px;margin: 265px 0 0 -898px;opacity: 0;"
    src="https://tieba.baidu.com/f?fr=wwwt&kw=%E7%BE%8E%E5%A5%B3"
  ></iframe>
</body>
```
通过设置ifame标签的透明度opacity就可以完成点击劫持。

## 3. 如何避免点击劫持事件
参考[点击劫持防御](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/clickJacking_defence_X-Frame-Options.md)。
