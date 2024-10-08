# **treeShaking是什么？**

Tree shaking 是一种用于移除 JavaScript 代码中未使用部分的优化技术。它主要依赖于静态分析和模块的特性，通常在构建过程中进行。

# treeShaking的原理是什么

Tree shaking 通过静态分析来确定哪些代码是未被使用的。流程如下。

**1.  静态导入和导出** ： ES6 模块系统允许使用 `import` 和 `export` 语句来明确地声明模块之间的依赖关系。

**2.  确定性分析** ： 构建工具在构建阶段可以通过分析模块的导入和导出语句来确定哪些代码路径是实际被使用的。

**3. 模块分析** ： 构建工具会遍历整个模块依赖树，从入口点开始，递归地分析每个模块的导入和导出，构建一个依赖树。

**4. 移除未使用代码：**

在确定哪些代码没有被使用后，构建工具会从最终的构建产物中移除这些未使用的代码。这一过程包括：

* 移除未使用的导入（imports）
* 移除未使用的导出（exports）
* 移除未使用的函数、变量、类等代码部分

**5. 副作用检测** ： 为了安全地移除未使用的代码，构建工具需要能够检测代码是否有副作用。如果一个模块或代码段有副作用（如修改全局状态、注册事件监听器等），那么即使它没有被直接引用，也不能被移除，因为它可能对应用程序的运行时行为有影响

**6. 限制**：动态导入和使用（如 `require` 或 `import()`）的代码通常无法被静态分析，因而无法被 Tree shaking。
