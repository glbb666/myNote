### path是什么

`path` 模块提供了一些实用工具，用于处理文件和目录的路径。 可以使用以下方式访问它：

```javascript
const path = require('path');
```

- `path.resolve()` 方法会将路径或路径片段的序列解析为绝对路径。

### path.resolve

- 没有参数

```javascript
path.resolve()
```

返回当前工作目录的绝对路径

- 有参数

  把给定的路径序列从右到左进行处理。后处理的（左边）会追加到先处理的（右边）前面，直到构建出相对路径。

  ```javascript
  path.resolve('/目录1/目录2', './目录3');
  ```

  如果在处理完所有给定的 `path` 片段之后**还未生成绝对路径，则会使用当前工作目录**。

  ```javascript
path.resolve('目录1', '目录2/目录3/', '../目录4/文件.gif');
  // 如果当前工作目录是 /目录A/目录B，
  // 则返回 '/目录A/目录B/目录1/目录2/目录4/文件.gif'
  ```
  
  生成的路径会被规范化，并且尾部的斜杠会被删除（除非路径被解析为根目录）。