
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [数字枚举](#数字枚举)
- [字符串枚举](#字符串枚举)
- [异构枚举（Heterogeneous enums）](#异构枚举heterogeneous-enums)
- [const枚举](#const枚举)

<!-- /code_chunk_output -->
### 数字枚举
```TypeScript
enum Direction {
    Up = 1,
    Down,
    Left,
    Right
}
```

### 字符串枚举
```TypeScript
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}
```

### 异构枚举（Heterogeneous enums）
```TypeScript
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}
```

### const枚举
为了避免在额外生成的代码上的开销和额外的非直接的对枚举成员的访问，我们可以使用 const枚举
```TypeScript
const enum Directions {
    Up,
    Down,
    Left,
    Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]

// 生成后的代码为：
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```