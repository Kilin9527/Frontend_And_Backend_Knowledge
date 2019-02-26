<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [Node中的module和require学习记录](#node中的module和require学习记录)
	* [一、module和require概述](#一-module和require概述)
	* [二、module和require详解](#二-module和require详解)
		* [1. 模块的加载顺序](#1-模块的加载顺序)
		* [2. 模块路径分析和文件定位](#2-模块路径分析和文件定位)
			* [路径分析：相对路径文件定位](#路径分析相对路径文件定位)
			* [路径分析：无路径文件定位](#路径分析无路径文件定位)
		* [3. 模块编译](#3-模块编译)
			* [js文件编译](#js文件编译)
			* [node文件编译](#node文件编译)
			* [json文件编译。](#json文件编译)
	* [参考链接](#参考链接)

<!-- /code_chunk_output -->

# Node中的module和require学习记录

## 一、module和require概述
Node中的模块参考了CommonJS中的模块标准，主要分为三个部分，模块的定义，模块的引用和模块的标识。

**模块的定义**：
在CommonJS的规范中，每一个js文件就是一个独立的模块上下文，在这个模块上下文中创建的所有的属性、对象和方法都是私有的，每个模块上下文环境中有一个module对象，代表模块本身。

如果要导出属性、对象和方法，只需要将他们赋值给module.exports即可。例如下面的例子，test.js导出了一个方法someFunc，其他模块可以引用test.js模块，然后使用someFunc。
```javascript {.line-numbers}
// test.js
function someFunc() {
    console.log("123");
}

module.exports = someFunc;
```
**模块的引用**
在node中，模块只需要使用require方法，并传入一个标识符，node就会将该模块引入：
```javascript
var someFunc = require('test');
```
**模块的标识符**
模块标识符就是传递给require()方法的参数，它必须是小驼峰命名的字符串，或者是./ ../开头的相对路径，或者绝对路径。例如上一段的模块标识符就是*test*。

## 二、module和require详解
Node模块分为两大类别，核心模块和文件模块，如图：
<!-- ![Node_模块_1](../../assets/images/node/module&require/Node_Module&require.png) -->
![Node_模块_1](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/Node/module&require/Node_Module&require.png?raw=true)

**核心模块**：由官方提供的模块，代码被编译成了二进制代码，可以直接通过require获取到，例如：fs，http，net等。在node启动时，部分核心模块就被加载到内存当中，由于省略的文件定位和编译执行这两个步骤，所以核心模块的加载速度最快，如果其他非核心模块与核心模块名称冲突，会优先引用核心模块。
核心模块由三部分组成：
* buildin模块：Node中以C++形式提供的模块，为其他的js模块提供基础api，例如：tcp_wrap、contextify等。该模块会在node启动之前编译。
* constants模块：Node中的常量模块。
* native模块：Node中以javascript形式提供的模块，如fs，http，net等。有些native模块需要借助于builtin模块实现背后功能。

**文件模块**：一般是指第三方模块，需要在运行时加载，引入速度比核心模块要慢一些。

### 1. 模块的加载顺序
当用户在require()一个模块时，node是如何查找并加载该模块？

看看require的源代码：
```javascript {.line-numbers}
// Loads a module at the given file path. Returns that module's `exports` property.
Module.prototype.require = function(id) {
  validateString(id, 'id');
  if (id === '') {
    throw new ERR_INVALID_ARG_VALUE('id', id,
                                    'must be a non-empty string');
  }
  return Module._load(id, this, /* isMain */ false);
};
```
1. 检查参数类型，这里需要传入string类型的。
2. 检查参数是否为空。
3. 调用内部_load方法，将参数传递给它。

接着看看_load发生了什么，只截取关键部分代码：
```javascript {.line-numbers}
// Check the cache for the requested file.
// 1. If a module already exists in the cache: return its exports object.
// 2. If the module is native: call `NativeModule.require()` with the
//    filename and return the result.
// 3. Otherwise, create a new module for the file and save it to the cache.
//    Then have it load  the file contents before returning its exports
//    object.
Module._load = function(request, parent, isMain) {
  // Case 1
  var cachedModule = Module._cache[filename];
  if (cachedModule) {
    updateChildren(parent, cachedModule, true);
    return cachedModule.exports;
  }
  // Case 2
  if (NativeModule.canBeRequiredByUsers(filename)) {
    debug('load native module %s', request);
    return NativeModule.require(filename);
  }
  // Case 3
  var module = new Module(filename, parent);
  Module._cache[filename] = module;
  return module.exports;
};
```
1. 如果require的模块已经在缓存当中，直接读取该模块的导出内容。
2. 如果require的模块是原生模块，调用NativeModule.require()去请求该模块。
3. 如果require的不属于以上两种情况，则创建指定的模块并缓存。

再深入一下看看NativeModule.require()发生了什么，同样只截取关键代码：
```javascript {.line-numbers}
NativeModule.require = nativeModuleRequire;

function nativeModuleRequire(id) {
  const mod = NativeModule.map.get(id);
  if (mod.loaded || mod.loading) {
    return mod.exports;
  }

  moduleLoadList.push(`NativeModule ${id}`);
  mod.compile();
  return mod.exports;
}
```
1. 从moduleLoadList(原生模块的缓存数组)获取指定的原生模块，如果已经引用过，返回原生模块的exports。
2. 如果没有引用过，将原生模块放入moduleLoadList中，编译并返回原生模块的exports。

模块加载小结：
* node模块优先从缓存中加载。
* 第三方模块(文件模块)如果与核心模块重名，只会加载核心模块。
* 核心模块的加载速度比文件模块略快。

### 2. 模块路径分析和文件定位
上一节介绍了模块加载顺序，这一节来看看模块是如何定位的，先看一张图：
<!-- ![模块加载优先级](../../assets/images/node/module&require/Node_Module&require_Priority_1.png) -->
![模块加载优先级](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/Node/module&require/Node_Module&require_Priority_1.png?raw=true)
当require()一个模块的时候，会从缓存中查找模块，如果不存在，判断请求的模块是否是原生模块，如果不是原生模块，则会把请求的模块当成文件模块来加载，文件模块的加载需要进行路径分析和文件定位。

#### 路径分析：相对路径文件定位
1. require(x)的参数x如果是以 "/"， "./"， "../"开始，会根据x所在的父级模块，确定x的绝对路径。
2. 在x的绝对路径下，依次添加后缀名确定是否存在对应的文件，如果存在，返回该文件的路径，停止查找。添加后缀名顺序如下：
*/x
*/x.js
*/x.json
*/x.node
如果找不到文件，进行下一步查找。
3. 将X当成目录，依次查找下列类型的文件，只要有一个存在，就返回该文件，停止查找。
*/x/package.json (main字段的内容)
*/x/index.js
*/x/index.json
*/x/index.node
如果仍旧没有找到，则抛出一个的异常。

#### 路径分析：无路径文件定位
如果require(x)的参数x不包含任何路径，只有一个模块名字会根据x所在的父目录，查找/node_modules目录是否存在，如果不存在，去再上一级的目录中找/node_modules，直到找到/node_modules为止，例如：

在C:/node_project/src/main.js中require("test")，则会依次查找如下目录：
C:/node_project/src/node_modules/test
C:/node_project/node_modules/test
C:/node_modules/test

如果目录被找到，再按照上一小节中的[路径分析：相对路径文件定位](#路径分析相对路径文件定位)规则去查找模块的位置，如果找不到指定的目录，抛出一个异常。

### 3. 模块编译
当定位到模块之后，还需要将模块编辑之后才能使用。

在node中，每一个模块都是Module对象。Module对象针对不同扩展名的文件，使用不同的加载方式：
#### js文件编译
我们知道每个模块文件中存在require、exports、module这3个变量，但是它们在模块文件中并没有定义，那么它们从何而来呢？在Node的API文档中，每个模块中还有__filename，__dirname这两个变量，它们又从何而来？
其实在编译过程中，Node对获取的JavaScript文件内容进行头尾包装。在头部添加(function (exports, require, module, __filename, __dirname){我们的自定义脚本});
来看看源代码，只截取部分：
```javascript {.line-numbers}
Module.wrap = function(script) {
  return Module.wrapper[0] + script + Module.wrapper[1];
};

Module.wrapper = [
  '(function (exports, require, module, __filename, __dirname) { ',
  '\n});'
];

Module.prototype._compile = function(content, filename) {
  var wrapper = Module.wrap(content);

  var compiledWrapper = vm.runInThisContext(wrapper, ...);

  var result = compiledWrapper.call(thisValue, exports, require, module,filename, dirname);
  return result;
};
```
编译执行后，模块对象的exports属性被返回给调用方。exports属性上的任何方法和属性都可以被外部调用到，但是模块中的其余变量或属性则不可直接被调用。

#### node文件编译
这是用C/C++编写的扩展文件，通过process.dlopen()方法加载最后编译生成的文件。
实际上，.node的模块文件并不需要编译，因为它是C/C++源码编译生成的，dlopen()是跨平台的，在windows通过visualC++编译器编译生成，在nix通过gcc/g++编译器编译生成，.node文件在windows平台实际上是一个.dll文件，在nix平台是一个.so文件。
所以.node文件只有加载和执行的过程。在执行的过程中，模块的exports对象与.node模块产生联系，然后将exports对象返回给调用者。


#### json文件编译。
json文件的编译是3种文件模块编译方式中最简单的。Node利用fs模块同步读取JSON文件的内容之后，调用JSON.parse()方法得到对象，然后将它赋值给模块对象的exports，供外部模块调用。


## 参考链接
https://github.com/yjhjstz/deep-into-node/blob/master/chapter2/chapter2-2.md
https://cloud.tencent.com/developer/article/1005768