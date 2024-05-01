### Hook是什么

React的Hook是React 16.8中引入的新功能，它允许你在不编写类组件的情况下“钩入” React 的特性。常用的React Hook包括useState, useEffect, useContext等。

### 为什么使用Hook

1. 简化代码：Hook允许你在不使用类的情况下使用React的特性。
2. 可复用性：自定义Hook可以被多个组件复用，这减少了代码冗余和复杂性。
3. 清晰的逻辑：使用Hook，你可以按逻辑收集相关的代码，而不是按照生命周期方法分离它们。
4. 减少嵌套：Hook减少了高阶组件和render props模式的需要，这两者通常会导致组件树的深层嵌套。
5. 拥抱函数式编程：Hook鼓励你使用更多的函数式编程概念，比如纯函数和不可变性，这可以提高应用的稳定性和性能。

### **React 中提供的 hooks：**

#### 1. State Hooks

* useState
* useReducer：setState，useState 是该方法的封装

  ```js
  const [state, dispatch] = useReducer(reducer, initialState);
  ```

`useReducer` 和 `useContext` 经常一起使用，它们共同提供了一种在 React 应用中进行状态管理的强大模式，特别是当你需要在多个组件间共享状态时。这种模式可以是类似 Redux 状态管理的轻量级替代方案。这种组合的优势是

1. **维护性和可读性** ：`useReducer` 提供了将状态更新逻辑分离出来的能力，这意味着状态更新逻辑会集中在 reducer 函数中，而不是分散在各个组件的 `useState` 调用中。结合 `useContext` 使用可以进一步清晰地组织代码。
2. **性能优化** ：在大型应用中，状态更新可能会引起大面积的组件重新渲染。使用 `useContext` 配合 `useReducer` 可以通过 context 分发 `dispatch` 函数，从而确保只有需要响应状态改变的组件才会重新渲染。

#### 2.Effect Hooks

- useEffect:（有专门写一篇）

类似 componentDidMount/Update, componentWillUnmount，当效果为 componentDidMount/Update 时，总是在整个更新周期的最后（页面渲染完成后）才执行。

* `useLayoutEffect` 的时机则类似于 `componentWillMount`（在 React 17 及以前）和 `componentWillUpdate`，以及 `getSnapshotBeforeUpdate`。它在所有 DOM 变更之后立即同步运行，并在新的更新被绘制之前完成，这意味着你可以在渲染发生前同步修改 DOM。

#### 3. context Hooks

* useContext: context，需配合 createContext 使用

#### 4. ref Hooks（高级指引）

* useRef: ref
* useImperativeHandle: 给 ref 分配特定的属性

#### 5. 性能优化 Hook（Hook里有提到）

* useMemo: 可以对 setState 的优化
* useCallback: useMemo 的变形，对函数进行优化

### 注意

* 不能将 hooks 放在循环、条件语句或者嵌套方法内。hooks的底层实现机制是双向链表，react 是根据 hooks 出现顺序来记录对应状态的。
