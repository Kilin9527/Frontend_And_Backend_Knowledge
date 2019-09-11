
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [迭代器和生成器](#迭代器和生成器)
  - [可迭代性](#可迭代性)
  - [for..of 和 for..in 语句](#forof-和-forin-语句)

<!-- /code_chunk_output -->

### 迭代器和生成器

#### 可迭代性
当一个对象实现了Symbol.iterator属性时，我们认为它是可迭代的。
对象上的Symbol.iterator函数负责返回供迭代的值。

#### for..of 和 for..in 语句
* for..in迭代的是对象的键的列表。
* for..of迭代的是对象的键对应的值。
```Typescript
let someArray = ["A", "B", "C"];

for (let value of someArray) {
    console.log(value); // A, B, C
}

for (let key in someArray) {
    console.log(key); // 0, 1, 2
}
```
