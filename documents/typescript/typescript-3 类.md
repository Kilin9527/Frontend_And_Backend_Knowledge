
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
- [静态属性](#静态属性)
- [抽象类](#抽象类)

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

### 静态属性
用 static 定义的属性。同一个类的实体对象之间共享的属性。

### 抽象类
* 抽象类做为其它派生类的基类使用。
* 抽象类一般不会直接被实例化。
* 不同于接口，抽象类可以包含成员的实现细节。
* abstract关键字是用于定义抽象类和在抽象类内部定义抽象方法
* 抽象类中的抽象方法不包含具体实现并且必须在派生类中实现
```TypeScript
abstract class Department {

    constructor(public name: string) {
    }

    printName(): void {
        console.log('Department name: ' + this.name);
    }

    abstract printMeeting(): void; // 必须在派生类中实现
}

class AccountingDepartment extends Department {

    constructor() {
        super('Accounting and Auditing'); // 在派生类的构造函数中必须调用 super()
    }

    printMeeting(): void {
        console.log('The Accounting Department meets each Monday at 10am.');
    }

    generateReports(): void {
        console.log('Generating accounting reports...');
    }
}

let department: Department; // 允许创建一个对抽象类型的引用
department = new Department(); // 错误: 不能创建一个抽象类的实例
department = new AccountingDepartment(); // 允许对一个抽象子类进行实例化和赋值
department.printName();
department.printMeeting();
department.generateReports(); // 错误: 方法在声明的抽象类中不存在
```