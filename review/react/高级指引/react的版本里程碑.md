## React的里程碑：

### React 16.0

React在这个版本中引入了新的核心算法Fiber，这个算法使React能够更好地处理动画，布局和手势等异步UI任务。这也为未来的React功能打下了基础。

### React 16.3

这个版本中React对生命周期方法进行了更新。它引入了新的生命周期方法getDerivedStateFromProps和getSnapshotBeforeUpdate，同时弃用了componentWillMount，componentWillReceiveProps和componentWillUpdate这三个生命周期方法。

### React 16.6

这个版本中引入了一个新的静态方法getDerivedStateFromError和一个新的生命周期方法componentDidCatch，这两个方法都是为了更好地处理错误。

### React 16.8

这个版本是一个重大变革，因为它引入了Hooks。Hooks允许在不编写class的情况下使用state和其他React特性，这使得函数组件的能力大大增强。

在 React 16.8 之前，React 主要使用类组件开发复杂的业务，函数组件主要用于输出简单的UI。

Hooks 将函数组件的能力扩展到了类组件的水平，甚至更上一层楼，以下是一些重要的变化和区别：

1. 状态和生命周期：使用 `useState` 和 `useEffect` 等钩子，函数组件现在可以拥有和管理自己的状态，以及在组件的生命周期中执行副作用操作。
2. 代码重用：在引入 Hooks 之前，复用状态逻辑需要使用高阶组件（HOCs）或者渲染属性（render props）。Hooks 通过自定义 Hook 简化这一过程，使得状态逻辑更容易重用和封装。
3. 简化和减少嵌套：Hooks 减少了组件树中不必要的嵌套，因为状态和副作用可以直接在函数组件内部管理，而无需额外的组件或高阶组件。
4. 更好的副作用管理：`useEffect` 钩子为副作用（如数据获取、订阅或手动更改 DOM）提供了一个更加清晰和易于使用的 API。
5. 性能优化：Hooks 有助于避免不必要的组件重新渲染，因为使用 `useCallback` 和 `useMemo`，我们可以更轻松地控制依赖项和记忆化计算。

### React 17.0

虽然这个版本没有引入新的开发者可见的特性，但它为将来的React版本打下了基础，包括改进了事件处理。

事件处理的改动主要涉及到两个方面：

事件委托的更改：之前 React 会将所有事件处理器委托到 document 层，而在 React 17.0 中，React 选择将这些事件处理器委托到 root DOM 容器节点（即应用根节点，通常是 React 通过 ReactDOM.render() 渲染的那个容器）。这样做的好处是，如果有多个 React 版本共存，或者将 React 应用嵌入到非 React 网站中，不会互相影响事件处理。