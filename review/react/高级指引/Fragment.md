### 简介

众所周知，组件中只能有一个根元素，因此我们经常要为此向DOM中额外添加节点。`Fragments` 允许你将子列表分组，而无需向 DOM 添加额外节点。`key` 是唯一可以传递给 `Fragment` 的属性。

### 使用

可以从React中引入后使用

```jsx
import React, { Fragment } from 'react';
function ListItem({ item }) {
  return (
    <Fragment>
      <dt>{item.term}</dt>
      <dd>{item.description}</dd>
    </Fragment>
  );
}
```

可以直接使用

```jsx
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildC />
    </React.Fragment>
  );
}
```

还有一种新的短语法可以用来声明它，但是有一个缺点，不支持key

```jsx
class Columns extends React.Component {
  render() {
    return (
      <>
        <td>Hello</td>
        <td>World</td>
      </>
    );
  }
}
```

