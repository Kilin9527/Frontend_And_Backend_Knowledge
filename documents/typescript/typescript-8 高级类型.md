
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [交叉类型（Intersection Types）](#交叉类型intersection-types)
- [联合类型（Union Types）](#联合类型union-types)
- [类型保护与区分类型（Type Guards and Differentiating Types）](#类型保护与区分类型type-guards-and-differentiating-types)
  - [用户自定义的类型保护](#用户自定义的类型保护)
  - [typeof类型保护](#typeof类型保护)
  - [instanceof类型保护](#instanceof类型保护)
- [可以为null的类型](#可以为null的类型)
- [类型别名](#类型别名)
- [字符串字面量类型](#字符串字面量类型)
- [多态的 this类型](#多态的-this类型)
- [索引类型（Index types）](#索引类型index-types)
  - [索引类型和字符串索引签名](#索引类型和字符串索引签名)

<!-- /code_chunk_output -->
### 交叉类型（Intersection Types）
交叉类型是将多个类型合并为一个类型。

这让我们可以把现有的多种类型叠加到一起成为一种类型，它包含了所需的所有类型的特性。 例如， Person & Serializable & Loggable同时是 Person 和 Serializable 和 Loggable。这个类型的对象同时拥有了这三种类型的成员。
```TypeScript
function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U>{};
    for (let id in first) {
        (<any>result)[id] = (<any>first)[id];
    }
    for (let id in second) {
        if (!result.hasOwnProperty(id)) {
            (<any>result)[id] = (<any>second)[id];
        }
    }
    return result;
}

class Person {
    constructor(public name: string) { }
}
interface Loggable {
    log(): void;
}
class ConsoleLogger implements Loggable {
    log() {
        // ...
    }
}
var jim = extend(new Person("Jim"), new ConsoleLogger());
var n = jim.name;
jim.log();
```

### 联合类型（Union Types）
* 联合类型适合于那些值可以为不同类型的情况。
* 联合类型表示一个值可以是几种类型之一。
* 用竖线 "|" 分隔每个类型。
```TypeScript
function padLeft(value: string, padding: string | number) {
    // ...
}
```
如果一个值是联合类型，我们只能访问此联合类型的所有类型里共有的成员。
```TypeScript
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // okay
pet.swim();    // errors
```

### 类型保护与区分类型（Type Guards and Differentiating Types）
#### 用户自定义的类型保护
类型保护就是一些表达式，它们会在运行时检查以确保在某个作用域里的类型。 要定义一个类型保护，我们只要简单地定义一个函数，它的返回值是一个 类型谓词：
```TypeScript
function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim !== undefined;
}
```
在这个例子里， pet is Fish就是类型谓词。 谓词为 parameterName is Type这种形式， parameterName必须是来自于当前函数签名里的一个参数名。
```TypeScript
if (isFish(pet)) {
    pet.swim();
}
else {Frontend_And_Backend_Knowledge
    pet.fly();
}
```
注意TypeScript不仅知道在 if分支里 pet是 Fish类型； 它还清楚在 else分支里，一定 不是 Fish类型，一定是 Bird类型。

#### typeof类型保护
```TypeScript
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```
#### instanceof类型保护
instanceof类型保护是通过构造函数来细化类型的一种方式。
instanceof的右侧要求是一个构造函数，TypeScript将细化为：
* 此构造函数的 prototype 属性的类型，如果它的类型不为 any 的话
* 构造签名所返回的类型的联合

### 可以为null的类型

### 类型别名
```TypeScript
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === 'string') {
        return n;
    }
    else {
        return n();
    }
}
```
类型别名也可以是泛型:
```TypeScript
type Container<T> = { value: T };
```
类型别名可以在**属性里**引用自己:
```TypeScript
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}
```

### 字符串字面量类型
```TypeScript
// 你只能从三种允许的字符中选择其一来做为参数传递，传入其它值则会产生错误。
type Easing = "ease-in" | "ease-out" | "ease-in-out";
class UIElement {
    animate(dx: number, dy: number, easing: Easing) {
        if (easing === "ease-in") {
            // ...
        }
        else if (easing === "ease-out") {
        }
        else if (easing === "ease-in-out") {
        }
        else {
            // error! should not pass null or undefined.
        }
    }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // error: "uneasy" is not allowed here
```

### 多态的 this类型
```TypeScript
class BasicCalculator {
    public constructor(protected value: number = 0) { }
    public currentValue(): number {
        return this.value;
    }
    public add(operand: number): this {
        this.value += operand;
        return this;
    }
    public multiply(operand: number): this {
        this.value *= operand;
        return this;
    }
    // ... other operations go here ...
}

let v = new BasicCalculator(2)
            .multiply(5)
            .add(1)
            .currentValue();
```
继承之后的this:
```TypeScript
class ScientificCalculator extends BasicCalculator {
    public constructor(value = 0) {
        super(value);
    }
    public sin() {
        this.value = Math.sin(this.value);
        return this;
    }
    // ... other operations go here ...
}

let v = new ScientificCalculator(2)
        .multiply(5)
        .sin()
        .add(1)
        .currentValue();
```

### 索引类型（Index types）
使用索引类型，编译器能够检查使用了动态属性名的代码。
```TypeScript
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}

interface Person {
    name: string;
    age: number;
}
let person: Person = {
    name: 'Jarid',
    age: 35
};
let strings: string[] = pluck(person, ['name']); // ok, string[]
```
**keyof T**， 索引类型查询操作符。 对于任何类型 T， keyof T的结果为 T上已知的公共属性名的联合。例如:
```TypeScript
let personProps: keyof Person; // 'name' | 'age'
```
**T[K]**，索引访问操作符。

当你返回 T[K]的结果，编译器会实例化键的真实类型，因此 getProperty的返回值类型会随着你需要的属性改变。
```TypeScript
function getProperty<T, K extends keyof T>(o: T, name: K): T[K] {
    return o[name]; // o[name] is of type T[K]
}

let name: string = getProperty(person, 'name');
let age: number = getProperty(person, 'age');
let unknown = getProperty(person, 'unknown'); // error, 'unknown' is not in 'name' | 'age'
```

#### 索引类型和字符串索引签名
keyof和 T[K]与字符串索引签名进行交互。 如果你有一个带有字符串索引签名的类型，那么 keyof T会是 string。 并且 T[string]为索引签名的类型：
```TypeScript
interface Map<T> {
    [key: string]: T;
}
let keys: keyof Map<number>; // string
let value: Map<number>['foo']; // number
```
