### 一、state

`state`是在`constructor`中被初始化的。

`state`是组件私有的，它可以向下流动。`state`中数据的更新会被检测到，触发组件的`render`。

在class中添加一个如下的构造函数，就可以使用state了

```jsx
constructor(props) {
    super(props);//将 props 传递到父类的构造函数中
    this.state = {date: new Date()};//给state赋初值
}
```

直接修改`this.state.date`是不会触发render的，要想更新`state`中的数据，需要调用`this.setState`进行更新

### 二、setState

当你调用 `setState()` 的时候，React 会把你提供的对象**浅合并**到当前的 state。

`setState`有两种调用方式

- 传入对象，第一个参数是传入的对象，第二个是可选参数，可以传入一个回调，因为state的更新是异步的，所以回调中可以通过读取`this.state`获取到最新的`state`的值

```jsx
this.setState({counter: 1},()=>{
  console.log(this.state)
});
```

- 传入函数

⚠️**State 的更新可能是异步的**，出于性能考虑，React 可能会把多个 `setState()` 调用合并成一个调用。

因为 `this.props` 和 `this.state` 可能会异步更新，所以你不要依赖他们的值来更新下一个状态。

要解决这个问题，可以让 `setState()` 接收一个函数而不是一个对象。这个函数用上一个 state 作为第一个参数，将此次更新被应用时的 props 做为第二个参数，返回值就是要更新的state对象：

```jsx
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

⚠️注意：不能在`constructor`中使用`this.setState`，因为此时`state` 还没有进行初始化完毕。