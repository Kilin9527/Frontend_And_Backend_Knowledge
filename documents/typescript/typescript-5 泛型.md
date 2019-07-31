
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [泛型变量](#泛型变量)
- [泛型类型](#泛型类型)
- [泛型类](#泛型类)
- [泛型约束](#泛型约束)
- [在泛型约束中使用类型参数](#在泛型约束中使用类型参数)

<!-- /code_chunk_output -->
### 泛型变量
T

### 泛型类型
```TypeScript
function identity<T>(arg: T): T {
    return arg;
}

// Type1: 泛型函数的类型
let myIdentity: <T>(arg: T) => T = identity;

// Type2: 可以使用不同的泛型参数名，只要在数量上和使用方式上能对应上就可以。
let myIdentity: <U>(arg: U) => U = identity;

// Type3: 可以使用带有调用签名的对象字面量来定义泛型函数
let myIdentity: {<T>(arg: T): T} = identity;
```

可以根据第三条写一个接口
```TypeScript
interface GenericIdentityFn  {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

### 泛型类
```TypeScript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

### 泛型约束
* extends关键字来实现约束。
```TypeScript
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length); 
    return arg;
}
```

### 在泛型约束中使用类型参数
```TypeScript
function getProperty(obj: T, key: K) {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // okay
getProperty(x, "m"); // error: Argument of type 'm' isn't assignable to 'a' | 'b' | 'c' | 'd'.
```

### 在泛型里使用类类型
```TypeScript
function create<T>(c: {new(): T; }): T {
    return new c();
}
```