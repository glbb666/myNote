### 使用map构建组件列表

```jsx
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
  <li>{number}</li>
);
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

JSX 允许在大括号中[嵌入任何表达式](https://zh-hans.reactjs.org/docs/introducing-jsx.html#embedding-expressions-in-jsx)，所以我们可以内联 `map()` 返回的结果：

```jsx
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {
        numbers.map((number) =>
          <li>{number}</li>
      )}
    </ul>
  );
}
```

### key

**在 `map()` 方法中的元素需要设置 `key` 属性。**否则将会看到一个警告 `a key should be provided for list items`。

`key`用来帮助`React`标识元素的改变，提高渲染效率，是一个独一无二的**字符串**，一般用元素id，迫不得已可以使用元素索引index。

它不需要全局唯一，但在兄弟索引间唯一。

