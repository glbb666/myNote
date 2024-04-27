### useCallback

作用：用于缓存函数本身。

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

返回值：

- 返回一个缓存好的函数。

#### 什么情况下使用

1. 函数被传递给子组件：如果你需要传递一个内联回调函数给子组件并且依赖于某些值时，通常会用 `useCallback`来优化内联函数，函数引用不变，这样可以防止子组件渲染，优化性能

```jsx
const MyComponent = ({ value }) => {
  
  // 每次 MyComponent 渲染时都会创建一个新的函数实例
  const cachedCallback = () => {
    console.log(value);
  };

  // ...
};
```

为了避免这种情况，你可以使用 `useCallback`钩子。`useCallback`会在依赖项不变的情况下，跨多次渲染记住同一个函数实例，从而避免不必要的渲染。

2. **当函数创建的代价比较大时** ，如果你的函数执行了昂贵的计算或者在定义时进行了复杂的初始化，使用 `useCallback` 可以避免在每次渲染时都执行这些操作，从而节省资源。
3. **用作其他 Hooks 的依赖** ：某些自定义钩子可能依赖于稳定的函数引用来避免不必要的重计算或效果重新执行（例如，作为 `useEffect`, `useMemo`, 或其他自定义Hooks的依赖项）。
4. **回调引用传递给其他逻辑** ：在复杂的组件或是自定义Hooks中，可能需要传递回调到其他地方，而在传递过程中保持引用的不变性是重要的，以便于合理控制逻辑的执行时机。

#### 什么情况下没必要使用

`useCallback`的开销通常比创建一个新箭头函数的开销小，尤其是在函数体比较简单的情况下。但是，如果函数不是作为prop传递且组件渲染不频繁，或者该函数不是触发重渲染的因素，那么使用 `useCallback`所带来的优化可能是微不足道的。在这些情况下，额外使用 `useCallback`可能看起来没必要，因为它没有实质影响对于性能。

### useMemo

作用：用于缓存复杂计算的结果值。**主要用于避免在每次渲染时都执行计算开销大的函数。**

入参

- 内联回调函数
- 依赖数组

```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

返回值：

- 返回一个计算好的值。

### 它们的使用场景

而当你需要执行某个计算密集的操作并且想避免在每次渲染时都进行这个操作时，通常会用 `useMemo`来优化。

[什么时候使用 useMemo 和 useCallback](https://jancat.github.io/post/2019/translation-usememo-and-usecallback/?from=from_parent_mindnote)
