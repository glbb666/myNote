### async是什么

`async`是`generator`的语法糖。它和`generator`比有以下好处

- 内置执行器，`generator`返回的是一个`Iterator`对象，需用`next`来执行，而`generator`内置执行器，可以自动执行。

- 语义化，`generator`没有`async`语义化。`async`代表异步，`await`代表等待。
- 适用性，`yield`后面只能是`promise`对象或`Thunk`函数，`await`后面可以跟`promise`对象或者原始类型的值（数值、字符串和布尔值，但这时会自动转成立即 resolved 的 Promise 对象）。
- 返回值，`async`的返回值是一个`promise`，后续还可以进行链式操作。

