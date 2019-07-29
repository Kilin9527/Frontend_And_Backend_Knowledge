
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [推断类型](#推断类型)
- [可选参数](#可选参数)
- [默认参数](#默认参数)
- [剩余参数](#剩余参数)
- [this](#this)
- [重载](#重载)

<!-- /code_chunk_output -->

### 推断类型
TypeScipt可自动推断返回值类型

### 可选参数
* 参数名旁使用 ? 实现可选参数的功能。
* 可选参数必须跟在必须参数后面。

### 默认参数
* 在所有必须参数后面的带默认初始化的参数都是可选的，与可选参数一样，在调用函数的时候可以省略。
* 与普通可选参数不同的是，带默认值的参数不需要放在必须参数的后面。 如果带默认值的参数出现在必须参数前面，用户必须明确的传入 undefined值来获得默认值。

### 剩余参数
```TypeScript
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}
```
* 剩余参数会被当做个数不限的可选参数。
* 可以一个都没有，也可以有任意个。

### this


### 重载
* 查找重载列表，尝试使用第一个重载定义。 如果匹配的话就使用这个。因此，在定义重载的时候，一定要把最精确的定义放在最前面。