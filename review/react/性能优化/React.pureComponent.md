### `React.PureComponent`

`React.PureComponent` 继承自 [`React.Component`](https://zh-hans.reactjs.org/docs/react-api.html#reactcomponent) ，主要用来优化性能。

两者的区别在于

- `PureComponent` 中以浅比较 ` prop` 和 `state `的方式来实现了[`shouldComponentUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate)
- 而 `Component` 只要 `state `发生改变， 不论值是否与之前的相等，都会触发更新。

如果赋予 React 组件相同的 `props` 和 `state`，`render()` 函数会渲染相同的内容，那么在某些情况下使用 `React.PureComponent` 可提高性能。

### 什么是浅比较？

就是对比 `当前状态`和 `下一个状态`下的 `prop`和 `state`，比较**基本数据类型**是否相同（如： 'a' === 'a'）, 比较其引用地址是否相同。

> 注意：

#### 当比较数据是引用数据类型的情况

- 在 `PureComponent` 中，当对传入的对象或者数组进行直接赋值时，会改变值，但是因为并没有改变其引用地址，所以就不引起重新渲染。可以用拓展运算符处理这种情况，或者 `immutable`库。

### 组件渲染的情况

- 在React中，默认情况下，父组件的渲染将导致其所有子组件进行渲染流程（即尝试渲染）
- 如果子组件是普通组件，它将默认进行渲染，因为它的 `shouldComponentUpdate`默认返回 `true`。
- 如果子组件是纯组件，React提供的 `shouldComponentUpdate`将通过浅比较来检查props和state是否发生变化，如果没有变化，则不会进行实际的DOM更新。
- 父组件如果是纯组件，它的决定不更新（基于其自身的props和state浅比较）将会阻止子组件的渲染流程，除非子组件自身的state变化导致其重新渲染，这与父组件的纯组件特性无关。

### 总结

1. `PureComponent` 不仅会影响本身，而且会影响子组件，所以 `PureComponent`最好作为展示组件。
2. 如果 `prop `和 `state` 每次都会变，那么使用 `Component` 的效率会更好，因为浅比较也是需要时间的。
3. 若有 `shouldComponentUpdate`，则执行 `shouldComponentUpdate`，若没有 `shouldComponentUpdate`方法会判断是不是 `PureComponent`，若是，进行浅比较。在 `PureComponent`中使用 `shouldComponentUpdate`会有警告。

### 注意

> `React.PureComponent` 中的 `shouldComponentUpdate()` 仅作对象的浅层比较。仅在你的 props 和 state 较为简单时，才使用 `React.PureComponent`，或者在深层数据结构发生变化时调用 [`forceUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#forceupdate) 来确保组件被正确地更新。你也可以考虑使用 [immutable 对象](https://facebook.github.io/immutable-js/)加速嵌套数据的比较。
>
> 此外，`React.PureComponent` 中的 `shouldComponentUpdate()` 将跳过所有子组件树的 prop 更新。因此，请确保所有子组件也都是“纯”的组件
