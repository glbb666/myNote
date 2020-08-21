### useCallback

入参：

- 内联回调函数
- 依赖数组

```javascript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

出参：

- `memoized`回调函数

### useMemo

入参

- 内联回调函数
- 依赖数组

```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

出参：

- 返回一个 [memoized](https://en.wikipedia.org/wiki/Memoization) 值。

### 它们的使用场景

[什么时候使用 useMemo 和 useCallback](https://jancat.github.io/post/2019/translation-usememo-and-usecallback/?from=from_parent_mindnote)

