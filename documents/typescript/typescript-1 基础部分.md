
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [1. 字符串](#1-字符串)
* [2. 数组](#2-数组)
* [3. Any](#3-any)
* [4. 类型断言](#4-类型断言)
* [5. 对象的解构与重命名](#5-对象的解构与重命名)

<!-- /code_chunk_output -->

### 1. 字符串
模板字符串: 以( ` )包围字符内容，以${ expr }形式嵌入表达式。
```typescript
let name: string = `Gene`;
let sentence: string = `Hello, my name is ${ name }.`;
```
### 2. 数组
定义数组的方式有两种:
```typescript
let list: number[] = [1, 2, 3];
```
或
```typescript
let list: Array<number> = [1, 2, 3];
```
### 3. Any
有时候，我们会想要为那些在编程阶段还不清楚类型的变量指定一个类型。 这些值可能来自于动态的内容，比如来自用户输入或第三方代码库。 这种情况下，我们不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查。 那么我们可以使用 any类型来标记这些变量。
```typescript
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```
在对现有代码进行改写的时候，any类型是十分有用的，它允许你在编译时可选择地包含或移除类型检查。 你可能认为 Object有相似的作用，就像它在其它语言中那样。 但是 Object类型的变量只是允许你给它赋任意值 - 但是却不能够在它上面调用任意的方法，即便它真的有这些方法：
```typescript
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```
### 4. 类型断言
有时候你会遇到这样的情况，你会比TypeScript更了解某个值的详细信息。 通常这会发生在你清楚地知道一个实体具有比它现有类型更确切的类型。

通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。 TypeScript会假设你，程序员，已经进行了必须的检查。

类型断言有两种形式。 其一是“尖括号”语法：
```typescript
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;
```
另一个为as语法：
```typescript
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```
### 5. 对象的解构与重命名
可以给解构的属性进行重命名：
```typescript
let o = {
	a: "kilin",
	b: "Lee",
};
let { a, b } = o;
let { a: firstName, b: lastName} = o;
```