
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [模块](#模块)
  - [介绍](#介绍)
  - [导出](#导出)
    - [导出语句](#导出语句)
    - [重新导出](#重新导出)
  - [导入](#导入)
  - [export = 和 import = require()](#export-和-import-require)
  - [使用其它的JavaScript库](#使用其它的javascript库)
    - [外部模块](#外部模块)

<!-- /code_chunk_output -->
## 模块

### 介绍
从ECMAScript 2015开始，JavaScript引入了模块的概念。TypeScript也沿用这个概念。

* 在一个模块里的变量，函数，类等等在模块外部是不可见的，除非你明确地使用export形式之一导出它们。 
* 如果想使用其它模块导出的变量，函数，类，接口等的时候，你必须要导入它们，可以使用 import形式之一。
* 任何包含顶级import或者export的文件都被当成一个模块。
* 如果一个文件不带有顶级的import或者export声明，那么它的内容被视为全局可见的。

模块加载器: 在执行某个模块之前，查找并执行这个模块的所有依赖。
例如：CommonJS和Require.js都是模块加载器。

### 导出
任何声明（比如变量，函数，类，类型别名或接口）都能够通过添加export关键字来导出。

#### 导出语句
```Typescript
/* ZipCodeValidator.ts */
export interface StringValidator {
    isAcceptable(s: string): boolean;
}

export const numberRegexp = /^[0-9]+$/;

// ZipCodeValidator导出方式 1
export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
// ZipCodeValidator导出方式 2
export { ZipCodeValidator };
// ZipCodeValidator导出方式 3 - 重命名
export { ZipCodeValidator as mainValidator };
```
#### 重新导出
```Typescript
/* ParseIntBasedZipCodeValidator.ts */
// 导出原先的验证器但做了重命名
export { ZipCodeValidator as RegExpBasedZipCodeValidator } from "./ZipCodeValidator";
```

### 导入
```Typescript
// 导入内容
import { ZipCodeValidator } from "./ZipCodeValidator";
let myValidator = new ZipCodeValidator();

// 重命名导入的内容
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
let myValidator = new ZCV();
```

### export = 和 import = require()
CommonJS和AMD的环境里都有一个exports变量，这个变量包含了一个模块的所有导出内容。

CommonJS和AMD的exports都可以被赋值为一个对象, 这种情况下其作用就类似于 es6 语法里的默认导出，即 export default语法了。虽然作用相似，但是 export default 语法并不能兼容CommonJS和AMD的exports。

为了支持CommonJS和AMD的exports, TypeScript提供了export =语法。

export =语法定义一个模块的导出对象。 这里的对象一词指的是类，接口，命名空间，函数或枚举。

若使用export =导出一个模块，则必须使用TypeScript的特定语法import module = require("module")来导入此模块。

### 使用其它的JavaScript库
要想描述非TypeScript编写的类库的类型，我们需要声明类库所暴露出的API。
它们通常是在 .d.ts 文件里定义的。如果你熟悉C/C++，你可以把它们当做.h文件。

#### 外部模块
在Node.js里大部分工作是通过加载一个或多个模块实现的。 我们可以使用顶级的 export声明来为每个模块都定义一个.d.ts文件，但最好还是写在一个大的.d.ts文件里。 我们使用与构造一个外部命名空间相似的方法，但是这里使用 module关键字并且把名字用引号括起来，方便之后import。
```Typescript
/* node.d.ts */
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }

    export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
    export function normalize(p: string): string;
    export function join(...paths: any[]): string;
    export let sep: string;
}