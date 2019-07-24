
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [继承](#继承)
- [公共，私有与受保护的修饰符](#公共私有与受保护的修饰符)
  - [public - 默认访问级别](#public-默认访问级别)
  - [private](#private)
  - [protected](#protected)
- [readonly修饰符](#readonly修饰符)
  - [参数属性](#参数属性)
- [存取器 getters/setters](#存取器-getterssetters)

<!-- /code_chunk_output -->

### 继承
通过 extends 关键字继承。

派生类通常被称作 子类，基类通常被称作 超类。

派生类的构造函数必须调用 super()，它会执行基类的构造函数。而且，在构造函数里访问 this 属性之前，一定要调用 super()。这个是TypeScript强制执行的一条重要规则。

### 公共，私有与受保护的修饰符
#### public - 默认访问级别
在 TypeScript 里，成员都默认为 public。
#### private
当成员被标记成 private 时，它就不能在声明它的类的外部访问。
#### protected
protected 修饰符与 private 修饰符的行为很相似，但有一点不同， protected 成员在派生类中仍然可以访问。

### readonly修饰符
你可以使用 readonly关键字将属性设置为只读的。
只读属性必须在声明时或构造函数里被初始化。
#### 参数属性
参数属性通过给构造函数参数前面添加一个访问限定符来声明。
* public
* private
* protected
参数属性把声明和赋值合并至一处。
```TypeScript
class Octopus {
    readonly numberOfLegs: number = 8;
    constructor(readonly name: string) {
    }
}
```
Octopus拥有两个属性，分别是 numberOfLegs 和 name。

### 存取器 getters/setters
* 存取器要求你将编译器设置为输出ECMAScript 5或更高。
* 不支持降级到ECMAScript 3。
* 只带有 get不带有 set的存取器自动被推断为 readonly。