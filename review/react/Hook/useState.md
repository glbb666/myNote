### useState

- 基础用法

```javascript
  const [state, setState] = useState(initialState);
  setState(newState);
```

- 函数式更新

```javascript
  const [count, setCount] = useState(initialCount);
 	setCount(prevCount => prevCount - 1)
```

⚠️注意

- 如果你的更新函数返回值与当前 state 完全相同，则随后的重渲染会被完全跳过。`React`是使用`Object.is`比较算法来比较`state`的。

  > Object.is接受两个参数进行比较，和"==="类似。但是区别是特殊的数字判断。
  >
  > 对Object.is来说-0和-0，+0和+0，NaN和NaN，返回结果是true。
  >
  > 但对===而言，-0===+0返回为true，NaN===NaN返回false

- 与 class 组件中的 `setState` 方法不同，`useState` 不会自动合并更新对象。

- 惰性初始`state`：

  **`initialState` 参数只会在组件的初始渲染中起作用，后续渲染时会被忽略。**如果初始` state `需要通过复杂计算获得，则可以传入一个函数，在函数中计算并返回初始的 `state`，此函数只在初始渲染时被调用：

  ```javascript
  const [state, setState] = useState(() => {
    const initialState = someExpensiveComputation(props);
    return initialState;
  });
  ```

  

