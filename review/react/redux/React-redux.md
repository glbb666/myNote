## React-Redux

React-Redux 是 React 的官方 Redux 绑定库。它允许开发人员以更清晰和更组织的方式使用 React 组件连接到 Redux 状态管理库。

为什么需要 React-Redux 呢？在没有 React-Redux 的情况下，如果你想要在 React 应用程序中使用 Redux，你得自己手动订阅 store，并且管理在组件中的状态的更新。这会使得代码变得复杂，难以维护，也增加了出错的概率。

React-Redux 提供了两个核心功能：`Provider` 和 `connect`。

1. `Provider` 组件是用来包裹你的应用的。它把 Redux store 传递给应用里面的组件树。
2. `connect` 函数是用来将 React 组件连接到 Redux store 的。通过 `connect`，你可以订阅部分 store 的更新，并将状态映射到组件的 props 上。你也可以把 action creators 作为 props 传递给组件，使得你能够从组件内部 dispatch actions。

不仅如此，它还提供了 新的 Hooks API (`useDispatch` 和 `useSelector`)。

让我们看看它们的使用。

### Provider

创建你的 Redux store 并使用 `Provider` 组件包裹你的应用，这样 Redux store 就可以在所有的组件中访问了

```jsx
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import todos from './reducers';
import App from './App'; // 假设 App 是你的根组件

const store = createStore(todos);

const TodoApp = () => (
    <Provider store={store}>
        <App />
    </Provider>
);

export default TodoApp;

```

### useSelector、useDispatch

在函数组件中可以使用 `useSelector` 来选择 store 中的某部分数据，使用 `useDispatch` 来 dispatch actions。

```js
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment } from './actions';

const CounterComponent = () => {
    const count = useSelector((state) => state.counter);
    const dispatch = useDispatch();

    return (
        <div>
            <span>{count}</span>
            <button onClick={() => dispatch(increment())}>Increment</button>
        </div>
    );
};

export default CounterComponent;

```

### connect

函数组件可以使用Connect来访问 Redux store，类组件必须使用 `connect` 来访问 Redux store。

connect一般只会使用到前两个参数mapStateToProps和mapDispatchToProps。

```js
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { increment } from './actions';

class CounterComponent extends Component {
    render() {
        // 通过 this.props 访问到 mapStateToProps 和 mapDispatchToProps 提供的数据和方法
        return (
            <div>
                <span>{this.props.count}</span>
                <button onClick={this.props.increment}>Increment</button>
            </div>
        );
    }
}

const mapStateToProps = (state) => ({
    count: state.counter
});

const mapDispatchToProps = (dispatch) => ({
    increment: () => dispatch(increment())
});

// connect 方法将组件链接到 Redux store
export default connect(mapStateToProps, mapDispatchToProps)(CounterComponent);

```

`connect` 是 Redux 的一个高阶组件 (HOC)，主要用来将 Redux 中的 state 和 action 转换为 React 组件的 props。`connect` 函数可以接受四个参数，它们都是可选的。这些参数按顺序包括：

#### mapStateToProps

- `mapStateToProps` 是一个函数，它定义了如何将 Redux store 中的 state 映射到组件的 props。它接收整个 store 的 state 作为它的第一个参数，并返回一个对象，这个对象的键值对将会被合并到组件的 props 中。
- 如果 `mapStateToProps`为空（ 即设置为()=>({}) )，那组件里的任何更新都不会触发组件的 `render`方法

```jsx
const mapStateToProps = (state) => {
  return {
    // prop : state.xxx  | 意思是将state中的某个数据映射到props中
    foo: state.bar
  }
}
```

渲染的时候就可以使用 `this.props.foo`

```jsx
class App extends Component {
    constructor(props){
        super(props);
    }
    render(){
        return(
        	// 这样子渲染的其实就是state.bar的数据了
            <div>this.props.foo</div>
        )
    }
}
export default connect(mapStateToProps)(App);
```

#### mapDispatchToprops

- 可以是一个对象，也可以是一个函数（负责输出）
- 作用：将 `dispatch(action)`绑定到组件的 `props`上，这样组件就可以派发 `Action`，更新 `state`了

##### `object`型 `mapDispatchToprops`

当 `mapDispatchToProps` 是一个对象时，这个对象内部的每一个字段都是一个 action creator（即创建 action 的函数）

React-Redux 会自动调用 `bindActionCreators` 来将 `mapDispatchToProps`的每个 action creator 包装成一个新的函数。这个新的函数会调用action creator 创建一个 action 对象，并将其 dispatch 到 store。使用对象形式可能是最简单直接的方式，尤其是当你不需要在 dispatch action 时添加额外的逻辑或参数时。

例如：

```js
// Action creator
const addTodo = (text) => ({
    type: 'ADD_TODO',
    text
});

// mapDispatchToProps as an object
const mapDispatchToProps = {
    addTodo
};

export default connect(null, mapDispatchToProps)(TodoApp);

```

当你将上面的 `mapDispatchToProps` 对象传给 `connect`，`TodoApp` 组件的 props 会包含一个 `addTodo` 函数，当调用这个函数时，`addTodo` action creator 会被执行，结果 action 会被自动 dispatched。

##### `function`型 `mapDispatchToprops`

参数是 `dispatch`方法，这种方式更灵活，因为你可以添加额外的逻辑或基于 props 来定制 dispatch 的行为。

```jsx
// Action creator
const addTodo = (text) => ({
    type: 'ADD_TODO',
    text
});

// mapDispatchToProps as a function
const mapDispatchToProps = (dispatch) => {
    return {
        addTodoWithDispatch: (text) => dispatch(addTodo(text))
        // 可以添加更多 logic 或者基于 props 的处理
    };
};

export default connect(null, mapDispatchToProps)(TodoApp);

```

##### 对象型和函数型 `mapDispatchToprops`的区别

- 对象形式的 `mapDispatchToProps` 在使用时最为简单，因为你只需要提供一个由 action creators 组成的对象。当你在组件中调用你的这些 action creators 作为函数时，React-Redux会创建 action 并自动 dispatch 它们。
- 函数形式的 `mapDispatchToProps` 提供了更多的灵活性。你手动处理 `dispatch` 和 action creators 的结合，并且可以编写任何自定义的逻辑。这在你需要在 dispatch action 之前执行某些操作（如条件判断，日志记录，修改 action payload 等）时非常有用。函数形式也允许你访问组件的 props，因为 `mapDispatchToProps` 的第二个参数是 `ownProps`，它可以让你根据组件的 props 来定制 dispatch 的行为。

#### mergeProps

`mergeProps` 是 `connect` 函数的第三个可选参数，用于自定义如何将 `mapStateToProps`、`mapDispatchToProps` 以及组件自身的 props（即 `ownProps`）合并到一起。其实现细节可充分定制，因此它为你在 Redux 的 state、dispatched actions 和组件的 props 之间建立复杂关系提供了可能

```js
function mergeProps(stateProps, dispatchProps, ownProps)

```

* `stateProps`：由 `mapStateToProps` 返回的对象，反映了 Redux store 中所需的 state 片段。
* `dispatchProps`：由 `mapDispatchToProps` 返回的对象，包含了可以在组件中使用的已绑定的 action creators。
* `ownProps`：组件收到的 props。

##### 一些问题

dispatchProps 是一个由 mapDispatchToProps 函数返回的对象。可是mapDispatchToProps有可能是一个对象，这是不是意味着如果要用mergeProps，就必须使用函数式的mapDispatchToProps？

思考：不，当你提供一个对象作为 `mapDispatchToProps` 时，React-Redux 内部会使用 `bindActionCreators` 来把每个在对象中定义的 action creator 包装成可以直接被调用的函数。这个被包装后的对象会被作为dispatchProps。

##### `mergeProps` 的作用

`mergeProps` 函数的作用是将上述三个参数对象合并成一个新的对象，这个新对象代表了由 `connect` 包裹的 React 组件的最终 props。你可以在这个函数中进行逻辑判断，决定最终传递给包裹组件的 props。

##### 使用 `mergeProps` 的原因

* 当你想基于组件的 props 来选择不同的 Redux state 或 actions 时。
* 当上游的 props 更改可能导致组件不必要的重渲染时，你可以借助 `mergeProps` 来优化性能。
* 当你想把多个来源的 props 组合成特定形式，或者当你想把它们映射为完全不同的 props 时。

##### 对mergeProps的一些思考

可以在组件外部抽离一个业务组件，或者logic类，去处理一些逻辑或者model使用的问题，而不是在mergeProps中处理，这样其实会把逻辑和视图分离，同时能提供一定的复用性。

##### 一个 `mergeProps` 的例子

假设你有一个包含用户信息的 Redux state，你可能希望根据组件外部的 props 来决定是展示用户的名字还是邮箱。

```js
// 假设 state 结构如下
// state = { user: { name: 'Alice', email: 'alice@example.com' } }

const mapStateToProps = state => ({
  name: state.user.name,
  email: state.user.email
});

const mapDispatchToProps = dispatch => ({
  updateUser: (userData) => dispatch(updateUserAction(userData))
});

// 定义 mergeProps
const mergeProps = (stateProps, dispatchProps, ownProps) => ({
  // 原样传递 dispatchProps
  ...dispatchProps,
  
  // 根据 ownProps.decide 属性决定传递给组件的用户信息
  userInfo: ownProps.decide === 'name' ? stateProps.name : stateProps.email
});

// 使用 connect 连接 Redux 和组件
export default connect(
  mapStateToProps,
  mapDispatchToProps,
  mergeProps
)(UserComponent);

```

#### config

`connect` 函数的第四个参数是 `options` 配置对象，它允许你自定义 `connect` 函数的行为。这个对象中的各种字段可以让你控制 `connect` 的一些高级选项。

* `context`: 自定义组件应该从哪个 context 中获取 store。通常不需要指定这个参数，除非你在使用多个 Redux store 的情况下。
* `pure`: 如果为 `true`（默认值），`connect` 将假定组件是纯组件，并且基于 props 和 state 的浅对比，来阻止不必要的更新，类似于 `React.memo` 或 `shouldComponentUpdate`。这可以提高性能，但是要确保组件的 props 和 state 的对比结果是可靠的。
* `areStatesEqual`: 优化性能的一个选项，它允许你自定义比较下一次 state 和上一次 state 是否相等的逻辑，默认使用的是 `===` 浅对比。
* `areOwnPropsEqual`: 同样是性能优化，允许自定义比较下一次的 `ownProps` 和上一次的 `ownProps` 是否相等，默认也是使用 `===` 浅对比。
* `areStatePropsEqual`: 允许自定义比较下一次从 `mapStateToProps` 推导出的 props 和之前的 state props 是否相等，默认也是使用 `===` 浅对比。
* `areMergedPropsEqual`: 如果你定义了 `mergeProps` 函数，这个选项将允许你定义如何深入比较合并后的 props 是否相等，默认使用 `===` 浅对比。
* `forwardRef`: 如果为 `true`，`connect` 将向包裹的组件实例转发 `ref`。这可以让你访问到被 `connect` 包裹的组件实例。

这些配置选项让你有能力优化性能，特别是在性能问题复杂或者组件更新非常频繁的情况下。在大多数情况下，使用默认配置就已经足够了。

一个包含 `options` 参数的 `connect` 调用可能如下所示：

```js
import { connect } from 'react-redux';

const mapStateToProps = /* ... */;
const mapDispatchToProps = /* ... */;

const myConnect = connect(
    mapStateToProps,
    mapDispatchToProps,
    null, // mergeProps 如果你没有使用它的话
    {
        pure: true,
        areStatesEqual: (next, prev) => next === prev, // 自定义对比逻辑
        areOwnPropsEqual: (next, prev) => next === prev,
        areStatePropsEqual: (next, prev) => next === prev,
        areMergedPropsEqual: (next, prev) => next === prev,
        forwardRef: true,
    }
);

export default myConnect(MyComponent);

```

在这个例子中，如果你想自定义 props 的比较逻辑，可以提供自己的比较函数。例如，如果你的 props 或 state 结构非常复杂，可能需要自定义深度对比逻辑，或者在某些特殊情况下，你可能需要基于某些特别的逻辑来判断你的组件是否需要更新。

另外，如果你在使用 React 的新特性如 hooks，并且你想让父级组件能直接跨越 `connect` 获取到子组件的 ref，设置 `forwardRef: true` 就很有用。

通常，只有在出现性能问题时才需要调整这些选项，因为使用默认设置已经覆盖了大部分常见的使用场景。
