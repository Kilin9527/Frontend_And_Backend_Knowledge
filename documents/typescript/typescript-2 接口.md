
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [1. 接口初探](#1-接口初探)
* [2. 可选属性](#2-可选属性)
* [2. 只读属性](#2-只读属性)
	* [readonly vs const](#readonly-vs-const)
* [3. 额外的属性检查](#3-额外的属性检查)

<!-- /code_chunk_output -->
TypeScript的核心原则之一是对值所具有的结构进行类型检查。
在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。
### 1. 接口初探
```typescript
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```
LabelledValue接口就好比一个名字，它代表了有一个label属性且类型为string的对象。

需要注意的是，**我们在这里并不能像在其它语言里一样，说传给printLabel的对象实现了这个接口，我们只会去关注值的外形，只要传入的对象满足上面提到的必要条件，那么它就是被允许的**。

还有一点值得提的是，类型检查器不会去检查属性的顺序，只要相应的属性存在并且类型也是对的就可以。

### 2. 可选属性
带有可选属性的接口与普通的接口定义差不多，只是在可选属性名字定义的后面加一个?符号。
```typescript
interface SquareConfig {
  color?: string;
  width?: number;
}
```
### 2. 只读属性
带有可选属性的接口与普通的接口定义差不多，只是在可选属性名字定义的后面加一个?符号。
```typescript
interface Point {
    readonly x: number;
    readonly y: number;
}
```
#### readonly vs const
最简单判断该用readonly还是const的方法是看要把它做为变量使用还是做为一个属性。 做为变量使用的话用 const，若做为属性则使用readonly。
### 3. 额外的属性检查
如果直接传入的参数与类型不匹配，会被类型检查出，并报错。
```typescript
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```
注意传入createSquare的参数拼写为colour而不是color。

解决办法：
1. 给接口添加一个索引签名：
```typescript
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```
2. 使用类型断言：
```typescript
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```
3. 将传入的值放在对象当中：
```typescript
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```
