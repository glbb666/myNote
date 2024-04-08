## Redux的出现

 在 `redux`出现之前，`js`使用的是传统的 `mvc`框架。

 ![image-20200225190844079](images/image-20200225190844079.png)

在大型的 `js`应用中势必存在非常多的 `View`和 `Model`，且它们之间存在双向绑定的关系，如果处理不好就会出现混乱的绑定关系，对于项目的维护和问题的追踪都不友好。针对这种现象，`facebook`团队提出了单向数据流，禁止 `view`直接对话 `model`。

`Redux`是一种基于 `Flux`思想的具体实现，替你维护难以管理的 `state`，让 `state`的变化可控。

### Action

定义：描述已发生事件，能够携带数据的普通对象。

- 必须有 `type`属性，用于告知 `Reducer`发生了什么事
- `Payload`，存放一些数据，给 `Reducer`的处理做一些辅助的作用

```jsx
//例1
{
	type:'ADD_TODO',
	payload:{
		text:'do something'
	}
}
//例2
{
	type:'ADD_TODO',
	payload:new Error(),
	error:true
}
```

我们可以把 `action`写成函数形式去存放，把 `action`的 `type`抽离成大写常量形式存放

### Reducer

定义：`Reducer`是一个纯函数，执行计算，返回新的 `state`。

不要做一些副作用的动作，如打印时间，修改外部变量。

```jsx
(oldState,action) => newState
```

⚠️注意：

- 首次执行 `Redux`时，`Redux`自动执行一次 `Reducer`，此时 `state`是 `undefined`，我们应该初始化 `state`
- `Reducer`每次更新状态需要一个新的 `state`，因此不要修改旧的 `state`参数，而是将旧的 `state`参数做一个拷贝

```jsx
if(typeof state === 'undefined'){
  return initialState;
}
return {
  ...state,
  //更新state中的值
}
```

### Action和Reducer之间的关系

action没有直接dispatch到特定的reducer，而是dispatch到store，那么reducers是怎么接受到action的呢？

redux不会监听action。

已知store内部维护了一系列reducer和当前的应用状态state。

当action被dispatch到store之后，store会自动调用所有的reducer，并传入两个参数，当前的state和被呈递的action。当多个reducer需要响应同一个action时，它们各自独立处理自己的状态部分，并返回新的状态。redux会把这些返回的状态合并成一个新的状态。

问题：当多个reducer接收到actions，在reducer中处理的时候，会生成新的state，新的state会发生冲突吗？

不会，因为每个reducer管理一个state。这一切都依赖于辅助函数

### `combineReducers`

```jsx
import { combineReducers } from 'redux';

const rootReducer = combineReducers({
  user: userReducer,
  todos: todosReducer,
  // 可以继续添加其他 reducers
});
```

入参：一个对象，对象中的每个键都对应一个 reducer 函数，键名将决定状态树中的哪个部分会被该 reducer 函数管理。

返回值：`combineReducers` 创建一个新的 `rootReducer`，`rootReducer` 会把状态对象的 `user` 键的状态管理权交给 `userReducer`，把 `todos` 键的状态管理权交给 `todosReducer`。

新创建的 `rootReducer` 函数接受两个参数：当前状态树（state）和要处理的 action。它返回的是下一个状态树。

**根 reducer 函数的伪代码表示如下：**

```js
function rootReducer(state = {}, action) {
  return {
    user: userReducer(state.user, action),
    todos: todosReducer(state.todos, action),
    // 继续处理其他子状态片段
  };
}
```

这个过程确保每个 reducer 只处理状态树中它所负责的那部分，这允许开发者将 reducer 逻辑组织得更清晰、更模块化。每个 reducer 独立管理自己的状态片段，因此不会相互干扰。

最后，自动生成的 `rootReducer` 通常会被传递给 `createStore` 函数，以初始化整个 Redux store 的状态：

```js
const store = createStore(rootReducer);

```

知道了上述combineReducer的入参，出参和用法，接下来我们可以看看 `combineReducer`的实现。

```jsx
function combineReducers(reducers) {
  // 返回生成的根 reducer 函数
  return function combination(state = {}, action) {
    let hasChanged = false; // 标志整个 state 对象是否发生了变化

    // 使用 reduce 方法遍历 reducers 对象的 entries
    // 累加器 newState 作为新状态的容器，最终返回它
    const nextState = Object.entries(reducers).reduce((newState, [key, reducer]) => {
      // 提取对应 key 的旧状态
      const previousStateForKey = state[key];
      // 执行当前的 reducer，得到新的状态
      const nextStateForKey = reducer(previousStateForKey, action);

      // 将新状态保存到 newState 对象中
      newState[key] = nextStateForKey;
      // 通过比较新旧状态来决定是否发生了变化
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey;
  
      return newState; // 逐步构建并返回最终的新状态对象
    }, {});

    // 如果 reducer 的数量与 state 对象 key 的数量不匹配，
    // 或者通过上面的处理发现有变化，则整个状态对象都视为发生了变化
    const stateKeysLength = Object.keys(state).length;
    const reducersKeysLength = Object.keys(reducers).length;
    hasChanged = hasChanged || stateKeysLength !== reducersKeysLength;
  
    // 如果有变化则返回新的状态对象否则返回旧的状态对象
    return hasChanged ? nextState : state;
  };
}

```

这段代码做了什么？

- 它接收一个 `reducers` 对象作为参数
- 它返回一个新的 rootReducer（`combination` 函数），该函数将处理整个应用的状态（`state`）和动作（`action`）
- `combination` 函数内部，使用 `Object.entries` 和 `reduce` 方法遍历每个 reducer。每个派发的action都会导致每个state的重新计算。每个旧的state会被对应的reducer进行处理，根据action得到新的state对象。并合并处理后的结果来构建最终的状态对象（`nextState`）。
- `hasChanged` 变量跟踪状态是否已经发生了变化。
- 如果 `nextState` 与 `state` 不同，或者 `reducers` 对象的 reducers 数量和 `state` 对象中的 keys 数量不匹配（意味着可能有新的 reducers 被添加或移除了），则标记 `hasChanged` 为 `true`
- 根据 `hasChanged` 的值，如果有变化，返回 `nextState`；如果没有变化，则返回原先的 `state` 对象，确保了 state 的引用不变，优化了性能

rootReducer在创建store时通过 `createStore` 或 `configureStore`（如果使用 Redux Toolkit）传入。一旦 store 被创建，它会保持对这个传入的根 reducer 的引用。Store 使用这个根 reducer 来处理派发的 action，并计算新的 state。

#### **一些问题**

**rootReducer在什么情况下会发生变化？**

通常情况下，reducers 在应用的整个生命周期内都不会变。它们是在编写应用时静态定义的。不过，Redux Toolkit 和 Redux 提供了一	些增强的 API 支持 reducer 的动态注入或替换，比如 `replaceReducer` 方法。这通常在代码拆分和按需加载时使用，当新的代码块（包括新的 reducer）被下载并执行时，可能会动态添加到应用中。

**如果 reducers 发生变化，应该要生成新的 combination 吗？**

这通常在**代码拆分和异步加载 reducer 的情况下发生** —— 你需要使用 store 的 `replaceReducer` 方法来传入新的根 reducer。这样做会替换 store 中的现有 reducer，并且以后派发的 action 会使用这个新的 reducer 来计算 state。

**如果每次更新 reducers 都会生成新的 combination，为什么会存在 state 和 reducer 长度不一致的情况呢？**

这通常意味着存在配置错误。例如，可能在调用 `combineReducers` 时，传入了一个新的 reducer集合，但某些 reducer 缺失了初始状态。`combineReducers` 函数实现中检查 state 和 reducer 长度不一致的逻辑，主要是作为一种健壮性和正确性的保障，保证在这种意外情况发生时，我们能得到一个可预测和一致的 state 结构。

在某些高级用例或异步加载/code-splitting 的场景中，你可能会动态地注入新的 reducer 或者替换现有的 reducer。在这种情况下，你需要显式地告诉 store 使用新的 reducer 通过调用 `replaceReducer`。否则可能就会出现长度不一致的情况。

### Store

内部存储数据的仓库。

#### createStore

创建 `store`

```javascript
const store = createStore(reducer,[initialState],[enhancer]);
// initialState用来初始化state
// enhancer是高阶函数，用来增强Store
```

#### getState

获取 `store`里面存储的 `state`

```jsx
store.getState().changeNumber.number;
```

#### dispatch

派发 `Action`，通知 `Reducer`去更新 `state`

```jsx
store.dispatch(actions.number.incrementNum());
```

#### subscribe(listener)

- 注册回调函数，当 `state`发生变化时，会自动触发回调函数

```jsx
const update = ()=>{}//更新view
store.subscribe(update);
```

- 该方法的返回值也是一个函数对象，调用后可以取消注册的回调函数

```jsx
const update = ()=>{}//更新view
const cancelUpdate = store.subscribe(update);
<Button onClick={cancelUpdate}>unsubscribe</Button>
```

### 实例

1. 安装

```nginx
npm install redux -S // 安装
```

2. 引入

```javascript
//1. 
import { createStore } from 'redux'
import rootReducer from './reducers';//reducer在文件内可省略
```

3. 创建 `reducer`

   a. 可以直接在同一个文件中这么写

```javascript
const reducer = (state = {count: 0}, action)
  switch (action.type){
    case 'INCREASE': return {count: state.count + 1};
    case 'DECREASE': return {count: state.count - 1};
    default: return state;
  }
}
```

    b. 也可以创建一个`reducer`文件夹，在内部用 `combineReducers`把多个 `reducer`进行一个整合

```javascript
//reducers/index.js
import {combineReducers} from 'redux';
import users from './users';
import counter from './counter';
export default combineReducers({
  users,
  counter
})

//reducers/user.js
const users = (state={},action={})=>{
  switch (action.type){
    default:return state;
  }
}
export default users;

//reducer/counter.js
const counter = (state= 1 ,action={})=>{
  switch (action.type){
    default:return state;
  }
}
export default counter;
```

4. 创建 `actions`

```javascript
import {INCREASE,DECREASE} from './constants';
// 我们可以把大写常量放在constants中
const actions = {
  increase: () => ({type: INCREASE}),
  decrease: () => ({type: DECREASE})
}
```

5. 创建store

```javascript
const store = createStore(reducer);---------->⑶
store.subscribe(() =>
  console.log(store.getState())
);
store.dispatch(actions.increase()) // {count: 1}
store.dispatch(actions.increase()) // {count: 2}
store.dispatch(actions.increase()) // {count: 3}
```

### `Redux`三大原则

1. 单一数据源 `Store`
2. 只能通过 `Dispatch Action`来修改 `state`
3. 使用 `Reducer`纯函数来执行修改 `state`

使用 `Redux`有两个动作，一个是初始化，一个是更新。

先看初始化：

![image-20200225192618285](images/image-20200225192618285.png)

再看更新：

在新的网页中，用户会有非常多的交互行为，比如用户点击页面的按钮，去改变页面的数据。

![image-20200225194309967](images/image-20200225194309967.png)

用户点击，会调用 `store`的 `dispatch`方法， `dispatch`出一个 `action`由 `reducer`接受，`reducer`会根据 `action`生成一个新的 `state`，接着把它放到 `store`中存储。在初始化过程当中，注册在 `store`上的 `listener` 就会被一一调用和执行， 我们通常会在 `listener`中做视图的更新动作，并且在视图更新中获取 `store`的最新数据，通过获取和更新，用户就可以看到页面中数据的展示。

## React-Redux

React-Redux 是一个绑定库，建立在 Redux 之上，专门用于在 React 应用中将组件连接到 Redux store。

如果仅使用 React-Redux 而没有 Redux，那么实际上是无法进行数据管理的。

* 它提供了 `Provider` 组件和 `connect` 函数，以及新的 Hooks API (`useDispatch` 和 `useSelector`)。
* `Provider` 组件使得 Redux store 对整个应用中的组件树可用。
* `connect` 函数用于将 Redux state 映射到 React 组件的 props，并将 action dispatchers 传递给组件，这样组件就可以触发状态更新。
* React-Redux 的 Hooks API 提供了一个更现代、函数式的方法来让 React 组件访问 Redux store 的状态，并派发 actions。

#### `<Provider store>`

作用：`store`直接集成到 `React`应用的顶层 `props`里面，只要各个子组件能访问到顶层 `props`就行了

将入口组件包进去（被包进的组件及其子组件才能访问到 `Store`，才能使用 `connect`方法

```jsx
import { Provider } from 'react-redux';
render(
	<Provider store = {store}>
  	<App />
  </Provider>，
  document.getElementById('app')
)
```

#### Connect

被 `Provider`包裹的组件(例如上面的 `App`)内部就可以使用 `connect`

```javascript
connect([mapStateToProps],[mapDispatchToprops],[mergeProps],[options])(MyComponent)
```

##### mapStateToProps（输出函数）

- 作用：将 `<font color='red'>Store`里 `</font>`的 `state`变成 `<font color='red'>`组件的 `</font>props`。当 `state`更新时，会同步更新组件的 `props`，触发组件的 `render`方法。
- 如果 `mapStateToProps`为空（ 即设置为()=>({}) )，那组件里的任何更新都不会触发组件的 `render`方法

```jsx
const mapStateToProps = (state) => {
  return {
    // prop : state.xxx  | 意思是将state中的某个数据映射到props中
    foo: state.bar
  }
}
```

然后渲染的时候就可以使用 `this.props.foo`

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

##### mapDispatchToprops

- 可以是一个对象，也可以是一个函数（负责输出）
- 作用：将 `dispatch(action)`绑定到组件的 `props`上，这样组件就可以派发 `Action`，更新 `state`了

##### `object`型 `mapDispatchToprops`

- `key`是组件 `props`，`value`是 `Actor creator`

```jsx
const mapDispatchToProps = {
  incrementNum:action.number.incrementNum,
  decrementNum:action.number.decrementNum,
  clearNum:action.number.clearNum,
}
```

这样组件就可以通过 `this.props.incrementNum()`来派发 `action`

##### `function`型 `mapDispatchToprops`

- 是一个 `function`
- 参数是 `dispatch`方法
- 返回值是 `object`型 `mapDispatchToprops`

```jsx
import { bindActionCreators } from 'redux';
const mapDispatchToProps2 = (dispatch,ownProps)=>{
	return {
    incrementNum:bindActionCreators(action.number.incrementNum,dispatch),
    decrementNum:bindActionCreators(action.number.decrementNum,dispatch),
    clearNum:bindActionCreators(action.number.clearNum,dispatch),
  }
}
```

```jsx
class App extends Component {
    constructor(props){
        super(props);
    }
    componentDidMount() {
        this.props.clearNum();
    }
}
export default connect(mapStateToProps)(App);
```

##### mergeProps

经过 `connect`的 `props`有三个来源

- 由 `mapStateToProps`将 `state`映射的 `props`
- 由 `mapDispatchToprops`将 `dispatch(action)`映射的 `props`
- 组件自身的 `props`

`mergeProps`的参数对应了这三个来源，作用就是整合这三个来源（过滤，重新组织，根据 `ownProps`绑定不同的 `stateProps`和 `dispatchProps`

## redux-saga

如果按照原始的 `redux`工作流程，当组件中产生一个 `action`后会直接触发 `reducer`修改 `state`，`reducer`又是一个纯函数，也就是不能再 `reducer`中进行异步操作；

**而往往实际中，组件中发生的 `action`后，在进入 `reducer`之前需要完成一个异步任务,比如发送 `ajax`请求后拿到数据后，再进入 `reducer`,显然原生的 `redux`是不支持这种操作的**

这个时候急需一个中间件来处理这种业务场景，目前最优雅的处理方式自然就是 `redux-saga`
