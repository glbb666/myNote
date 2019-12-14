`addEventListener(event,function,useCaptrue)`

`addEventListener(event,function,options)`

- `event `指定事件名
- `function`指定事件发生后触发的函数
- `useCapture `指定事件是在冒泡还是捕获阶段执行，默认值`false`（在冒泡阶段执行）

- options 指定有关 `listener `属性的可选参数**对象** 

  可用的选项如下：

  - `capture`:  [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)，表示 `listener` 会在该类型的事件捕获阶段传播到该 `EventTarget` 时触发。
  - `once`:  [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)，表示 `listener 在添加之后最多只调用一次。如果是` `true，` `listener` 会在其被调用之后自动移除。
  - `passive`: [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)，设置为true时，表示 `listener` 永远不会调用 `preventDefault()。`

