
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [模块解析](#模块解析)
- [相对 vs. 非相对模块导入](#相对-vs-非相对模块导入)
- [模块解析策略](#模块解析策略)
  - [Classic](#classic)
  - [Node](#node)
    - [相对路径](#相对路径)
    - [非相对路径](#非相对路径)

<!-- /code_chunk_output -->
### 模块解析
模块解析是指编译器在查找导入模块内容时所遵循的流程。

### 相对 vs. 非相对模块导入
**相对导入**是以/，./或../开头的。
相对导入在解析时是相对于导入它的文件，并且不能解析为一个外部模块声明。你应该为你自己写的模块使用相对导入，这样能确保它们在运行时的相对位置。

**非相对的导入**是指除了相对导入之外的其他形式。
非相对模块的导入可以相对于baseUrl或通过路径映射来进行解析。它们还可以被解析成 外部模块声明。使用非相对路径来导入你的外部依赖。

### 模块解析策略
共有两种可用的模块解析策略：Node和Classic。 你可以使用 --moduleResolution标记来指定使用哪种模块解析策略。若未指定，那么在使用了 --module AMD | System | ES2015时的默认值为Classic，其它情况时则为Node。

#### Classic
这种策略在以前是TypeScript默认的解析策略。 现在，它存在的理由主要是为了向后兼容。

#### Node
##### 相对路径
假设有一个文件路径为 /root/src/moduleA.js，包含了一个导入var x = require("./moduleB"); Node.js以下面的顺序解析这个导入：
1. 检查/root/src/moduleB.js文件是否存在。
2. 检查/root/src/moduleB目录是否包含一个package.json文件，且package.json文件指定了一个"main"模块。 在我们的例子里，如果Node.js发现文件 /root/src/moduleB/package.json包含了{ "main": "lib/mainModule.js" }，那么Node.js会引用/root/src/moduleB/lib/mainModule.js。
3. 检查/root/src/moduleB目录是否包含一个index.js文件。 这个文件会被隐式地当作那个文件夹下的"main"模块。

##### 非相对路径