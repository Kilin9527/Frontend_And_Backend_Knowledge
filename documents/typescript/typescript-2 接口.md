
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. 接口初探](#1-接口初探)
- [2. 可选属性](#2-可选属性)
- [2. 只读属性](#2-只读属性)
  - [readonly vs const](#readonly-vs-const)
- [3. 额外的属性检查](#3-额外的属性检查)
- [函数类型](#函数类型)
- [可索引的类型](#可索引的类型)
- [类类型](#类类型)
- [继承接口](#继承接口)
- [混合类型](#混合类型)

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
### 函数类型
接口除了可以描述带有属性的普通对象外，也可以描述函数类型。

它就像是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。
```typescript
// 定义
interface SearchFunc{
    (source: string, subString: string): boolean;
}

// 使用
let mySearch: SearchFunc;
mySearch = function(src, sub) {
  let result = src.search(sub);
  return result > -1;
}
```
函数的参数名可以不与接口里定义的名字相匹配。
函数的参数类型可以不与接口里定义的类型相匹配，TypeScript的类型系统会推断出参数类型。
函数的返回值类型也可以不与接口里定义的类型相匹配，TypeScript的类型系统会推断出返回值类型。

### 可索引的类型
TypeScript支持两种索引签名：字符串和数字。
可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型。 

### 类类型
TypeScript用接口来明确的强制一个类去符合某种契约。

### 继承接口
接口可以相互继承。

一个接口可以继承多个接口，创建出多个接口的合成接口。

### 混合类型
接口能够描述JavaScript里丰富的类型。 因为JavaScript其动态灵活的特点，有时你会希望一个对象可以同时做为函数和对象使用，并带有额外的属性。
```typescript
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
// 作为函数使用
c(10);
// 作为对象使用
c.reset();
// 并带有额外的属性
c.interval = 5.0;
```

### 接口继承类
无法理解该部分内容。