React 的响应式数据处理是通过**虚拟 DOM** 和 **state** 的管理机制来实现的。核心思想是当组件的状态或属性发生变化时，React 会自动更新并重新渲染受影响的部分，而不需要手动操作 DOM。以下是其基本原理：

### 1. **虚拟 DOM & DIFF**

虚拟DOM的概念。当需要进行更新的时候不会直接去渲染，而是通过虚拟DOM的创建，虚拟DOM树的diff找出差异。

### 2. **React Fiber**

虚拟DOM只是一个抽象的概念。

React的Fiber架构，包含了虚拟DOM的具体实现。通过Fiber节点进行小范围渲染，提升 React 的响应能力。

Fiber

### 3. 协调

当在组件内产生差异的时候，update会首先被推入组件对应的Fiber节点的updateQueue链表，并且根据update的优先级产生一个过期时间。

同时，每次触发render的时候


在差异算法识别出需要更新的地方后，React 会进行协调过程，将差异更新到实际的 DOM 中。这个过程只会修改那些有变化的部分，尽量减少操作真实 DOM 的次数，提高性能。

### 4. **State 和 Props 驱动的渲染**

* **State** 是组件内部的数据，通过 `setState` 函数进行更新。当组件的 state 发生变化时，React 会触发重新渲染。
* **Props** 是由父组件传递给子组件的数据。当父组件的 props 变化时，子组件也会自动更新和重新渲染。

### 5. **组件生命周期**

React 组件有一套完整的生命周期方法，如 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount`。这些方法允许开发者在组件挂载、更新和卸载时执行特定的逻辑，进一步控制响应式数据的行为。

我有一个问题，当在组件内产生差异的时候，update会首先被推入组件对应的Fiber节点的updateQueue链表，并且根据update的优先级产生一个过期时间，接着这个时间会和render的期待时间进行比较，把在render时间之前的update拿出链表，对旧的state进行更新。我的问题是render的期待时间是怎么产生的。
