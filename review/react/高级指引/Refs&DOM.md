### 何时使用refs

当需要在典型数据流之外强制修改子组件。

- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。

避免使用 refs 来做任何可以通过声明式实现来完成的事情。

举个例子，避免在 `Dialog` 组件里暴露 `open()` 和 `close()` 方法，最好传递 `isOpen` 属性。

### 使用ref的两种方式

### 1.React.createRef

 使用 `React.createRef()` 创建ref，并通过 `ref` 属性附加到 React 元素。

对该节点的引用可以在 ref 的 `current` 属性中被访问。

`ref.current` 的值根据节点的类型而有所不同：

- 当 `ref` 属性用于 HTML 元素时，为底层 DOM 元素
- 当 `ref` 属性用于自定义 class 组件时，为组件的挂载实例
- **不能在函数组件上使用 `ref` 属性**，因为他们没有实例。但你可以在函数组件中使用 `ref`，只要它指向一个 DOM 元素或 class 组件

##### 为 DOM 元素添加 ref

```jsx
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个 ref 来存储 textInput 的 DOM 元素
    this.textInput = React.createRef();
  }
  focusTextInput = () => {
    this.textInput.current.focus();
  }
  render() {
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} />
        <input
          type="button"
          onClick={this.focusTextInput}/>
      </div>
    );
  }
}
```

React 会在组件挂载时给 `current` 属性传入 DOM 元素，并在组件卸载时传入 `null` 值。`ref` 会在 `componentDidMount` 或 `componentDidUpdate` 生命周期钩子**触发前**更新。

##### 为 class 组件添加 ref

如果我们想包装上面的 `CustomTextInput`，来模拟它挂载之后立即被点击的操作，我们可以使用 ref 来获取这个自定义的 input 组件并手动调用它的 `focusTextInput` 方法：

```jsx
class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }
  componentDidMount() {
    this.textInput.current.focusTextInput();
  }
  render() {
    return (
      <CustomTextInput ref={this.textInput} />
    );
  }
}
```

### 2.回调 Refs

传递给组件的ref属性值是一个函数，React 将在组件挂载时，会调用 `ref` 回调函数并传入 DOM 元素，当卸载时调用它并传入 `null`。在 `componentDidMount` 或 `componentDidUpdate` 触发前，React 会保证 refs 一定是最新的。这个函数中直接接受 React 组件实例或 HTML DOM 元素作为参数，不需要读取current属性值来获取。

```jsx
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = null;
    this.setTextInputRef = element => {
      this.textInput = element;
    };
    this.focusTextInput = () => {
      if (this.textInput) this.textInput.focus();
    };
  }

  componentDidMount() {
    // 组件挂载后，让文本框自动获得焦点
    this.focusTextInput();
  }

  render() {
    return (
      <div>
        <input
          type="text"
          ref={this.setTextInputRef}
        />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

⚠️注意：当函数是一个内联函数时，会执行两次，第一次传入null，第二次传入实例对象。

### 将 DOM Refs 暴露给父组件

你可能希望在父组件中引用子节点的 DOM 节点。通常不建议这样做，因为它会打破组件的封装，但它偶尔可用于**触发焦点或测量子 DOM 节点的大小或位置**。

如果你使用 16.3 或更高版本的 React, 这种情况下我们推荐使用 [ref 转发](https://zh-hans.reactjs.org/docs/forwarding-refs.html)。**Ref 转发是一个可选特性，其允许某些组件使用 `React.forwardRef` 接收 `ref`，并将其向下传递（换句话说，“转发”它）给子组件。**

- 转发 refs 到 DOM 组件

```jsx
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

1. 我们通过调用 React.createRef 创建了一个 React ref 并将其赋值给 ref 变量。
2. 我们通过指定 ref 为 JSX 属性，将其向下传递给 <FancyButton ref={ref}>。
3. React 传递 ref 给 forwardRef 内函数 (props, ref) => ...，作为其第二个参数。
4. 我们向下转发该 ref 参数到 <button ref={ref}>，将其指定为 JSX 属性。
5. 当 ref 挂载完成，ref.current 将指向 <button> DOM 节点。

> Ref 转发不仅限于 DOM 组件，你也可以转发 refs 到 class 组件实例中。

- 在高阶组件中转发refs

```jsx
function logProps(Component) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      const {forwardedRef, ...rest} = this.props;

      // 将自定义的 prop 属性 “forwardedRef” 定义为 ref
      return <Component ref={forwardedRef} {...rest} />;
    }
  }

  // 注意 React.forwardRef 回调的第二个参数 “ref”。
  // 我们可以将其作为常规 prop 属性传递给 LogProps，例如 “forwardedRef”
  // 然后它就可以被挂载到被 LogProps 包裹的子组件上。
  return React.forwardRef((props, ref) => {
    return <LogProps {...props} forwardedRef={ref} />;
  });
}
```

如果你使用 16.2 或更低版本的 React，或者你需要比 ref 转发更高的灵活性，你可以使用[这个替代方案](https://gist.github.com/gaearon/1a018a023347fe1c2476073330cc5509)将 ref 作为特殊名字的 prop 直接传递。

