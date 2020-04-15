


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [DOM](#dom)
- [Shadow Dom](#shadow-dom)
- [创建 Shadow Dom](#创建-shadow-dom)
- [只有部分标签支持shadow dom](#只有部分标签支持shadow-dom)
- [在 shadow dom 中修改宿主的样式](#在-shadow-dom-中修改宿主的样式)
- [shadow dom 中的 template](#shadow-dom-中的-template)
- [基于上下文的样式](#基于上下文的样式)
- [显示宿主的内容 - slot](#显示宿主的内容-slot)
- [总结](#总结)

<!-- /code_chunk_output -->

## DOM
DOM是文档对象模型，它将HTML视为一棵树，并提供一系列 API，javascript 等脚本语言通过 API 对其进行操作。

## Shadow Dom
Shadow Dom 属于 Web Components 规范的一个子集，主要用途是封装DOM树，正常的Dom，其js和css容易受到别的代码的干扰，但shadow dom规定只有其宿主才可以定义其表现，外部API无法获取shadow dom中的内容。 

优点：
* 封装性好，与外部隔离，shadow dom 外部的代码无法影响内部，同样，内部的代码也无法干扰外部。
* 代码可复用。

## 创建 Shadow Dom
可以通过 attachShadow() 方法创建 shadow dom。
```Typescript
// 获取 shadow dom 的宿主
var host = document.getElementById('a');
// 替换宿主的内容
var shadowRoot = host.attachShadow({mode:'open'});
// 给 shadow dom 添加内容
shadowRoot.innerHTML=`
    <p>这里是内容</p>
    <p>这里是内容</p>
`
```
* shadow dom需要一个宿主才能生效，宿主元素的内容将被隐藏，而shadow dom的内容将展示出来。
* shadow dom通过 attachShadow({ mode : 'open' | 'closed' }) 方法绑定到宿主。

如果 mode 设置为 open，允许宿主通过 shadowRoot 访问shadow dom的元素。
```Typescript
host.shadowRoot.querySelector('p').innerText = 'new text';
host.shadowRoot.querySelector('p').style.color = 'red';
```

如果 mode 设置为 closed，不允许访问shadow dom的元素。
```Typescript
// => TypeError: Cannot read property 'querySelector' of null 
host.shadowRoot.querySelector('p').innerText = 'new text';
```


## 只有部分标签支持shadow dom
```Typescript
+----------------+----------------+----------------+
|    article     |      aside     |   blockquote   |
+----------------+----------------+----------------+
|     body       |       div      |     footer     |
+----------------+----------------+----------------+
|      h1        |       h2       |       h3       |
+----------------+----------------+----------------+
|      h4        |       h5       |       h6       |
+----------------+----------------+----------------+
|    header      |      main      |      nav       |
+----------------+----------------+----------------+
|      p         |     section    |      span      |
+----------------+----------------+----------------+
```

## 在 shadow dom 中修改宿主的样式
```Typescript
<style>
    :host{
        color: red;
    }

    :host(:focus) {
        color: blue;
    }

    :host(.someClass) {
        color: yellow;
    }
</style>
```

## shadow dom 中的 template
shadow dom 规定了一种名为 template 的标签，它不会被解析为 dom 树的一部分，template 的内容可以被塞入到 shadow dom 中并且反复利用。
```Typescript
// HTML
<div id="host">
    123
</div>
   
<template id="template">
    <span>hello world</span>
</template>

<script>
var host = document.querySelector('#host');
var root = host.attachShadow({mode:'open'});
var template = document.getElementById("template").content.cloneNode(true);
root.appendChild(template);
</script>
```
注意 document.getElementById("template").content 中的 content 属性，它是 template 标签的特有属性。

## 基于上下文的样式
要选择特定祖先内部的host ，可以用:host-context()伪类函数。
```Typescript
// 只有当它是.main的后代时，此CSS代码才会选择shadow host ：
:host-context(.main) {
  font-weight: bold;
}

// 只有这种情况下好用
<body class="main">
  <div id="host">
  </div>
</body>
```

## 显示宿主的内容 - slot
如果shadow dom需要显示宿主的内容，可以使用slot标签。
首先在宿主中将需要显示的标签标记slot。
```Typescript
<div class="host">
  <p slot="content1">start</p>
  <p slot="content2">end</p>
</div>
```
然后在 shadow dom 中设置 slot元素。
```Typescript
shadowRoot.innerHTML = `
    <p>Kilin</p>
    <slot name='content1'></slot>
    <p>Lee</p>
    <slot name='content2'></slot>
`;
```
这样，宿主的元素就可以在 shadow dom 中显示出来了。

## 总结
通过shadow dom的学习，了解的shadow dom的机制，template特有的属性content，slot标签的使用。