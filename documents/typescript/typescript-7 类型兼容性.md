
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [类型兼容性](#类型兼容性)
- [比较两个函数](#比较两个函数)
  - [可选参数及剩余参数](#可选参数及剩余参数)
  - [函数重载](#函数重载)
- [类](#类)
  - [类的私有成员和受保护成员](#类的私有成员和受保护成员)

<!-- /code_chunk_output -->

### 类型兼容性
TypeScript结构化类型系统的基本规则是，如果x要兼容y，那么y至少具有与x相同的属性。
```TypeScript
interface Named {
    name: string;
}

let x: Named;
// y's inferred type is { name: string; location: string; }
let y = { name: 'Alice', location: 'Seattle' };
x = y;
```

这里要检查y是否能赋值给x，编译器检查x中的每个属性，看是否能在y中也找到对应属性。 在这个例子中，y必须包含名字是name的string类型成员。y满足条件，因此赋值正确。

检查函数参数时使用相同的规则：
```TypeScript
function greet(n: Named) {
    console.log('Hello, ' + n.name);
}
greet(y); // OK
```

### 比较两个函数
```TypeScript
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```
要查看x是否能赋值给y，首先看它们的参数列表。 x的每个参数必须能在y里找到对应类型的参数。 注意的是参数的名字相同与否无所谓，只看它们的类型。 这里，x的每个参数在y中都能找到对应的参数，所以允许赋值。

第二个赋值错误，因为y有个必需的第二个参数，但是x并没有，所以不允许赋值。

下面来看看如何处理返回值类型，创建两个仅是返回值类型不同的函数：
```TypeScript
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // OK
y = x; // Error, because x() lacks a location property
```
类型系统强制源函数的返回值类型必须是目标函数返回值类型的子类型。

#### 可选参数及剩余参数
比较函数兼容性的时候，可选参数与必须参数是可互换的。
源类型上有额外的可选参数不是错误，目标类型的可选参数在源类型里没有对应的参数也不是错误。

#### 函数重载
对于有重载的函数，源函数的每个重载都要在目标函数上找到对应的函数签名。 这确保了目标函数可以在所有源函数可调用的地方调用。

### 类
* 类有静态部分和实例部分的类型。
* 比较两个类类型的对象时，只有实例的成员会被比较。
* 静态成员和构造函数不在比较的范围内。
```TypeScript
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { }
}

class Size {
    feet: number;
    constructor(numFeet: number) { }
}

let a: Animal;
let s: Size;

a = s;  // OK
s = a;  // OK
```
#### 类的私有成员和受保护成员
类的私有成员和受保护成员会影响兼容性。 当检查类实例的兼容时，如果目标类型包含一个私有成员，那么源类型必须包含来自同一个类的这个私有成员。 同样地，这条规则也适用于包含受保护成员实例的类型检查。 这允许子类赋值给父类，但是不能赋值给其它有同样类型的类。
