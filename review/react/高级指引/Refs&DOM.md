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
2. 我们通过指定 ref 为 JSX 属性，将其向下传递给 `<FancyButton ref={ref}>`。
3. React 传递 ref 给 forwardRef 内函数 (props, ref) => ...，作为其第二个参数。
4. 我们向下转发该 ref 参数到 `<button ref={ref}>`，将其指定为 JSX 属性。
5. 当 ref 挂载完成，ref.current 将指向 `<button>` DOM 节点。

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

### ref在不同情况下的使用

- DOM元素：指向的是真实的DOM节点
- 类组件：指向的是组件的实例，可以调用组件的方法或访问实例属性。
- 函数组件：不能直接使用ref的，因为他们没有实例。但是，可以通过使用React.forwardRef API或useImperativeHandle hook配合useRef来使得函数组件可以接受ref。

函数组件没有ref，需要用forwardRef的方式转发从外层获取的ref，外层获取的ref一般是用React.createRef创建。

⚠️：调用函数组件时不能用回调ref。因为回调ref本质上还是拿的组件自身的ref，而函数组件没有ref。

forwardRef有两种使用方法

- const RefComponent = forwardRef((props,ref)=>return Component)，这里的ref就是从外部传入的，
- export forwardRef(Component)，这里外部传入的ref可以用useImperativeHandle(ref)的第一个参数接收到

### 实践

- 如果对函数组件使用ref的意图是获取根节点的状态，可以用方法一接受ref，把ref放在根节点上（注意，根节点是dom节点，这里就直接用了dom节点的ref机制）
- 如果对函数组件使用ref的意图是通过ref控制组件的自定义方法，那么forwardRef就需要结合useImperativeHandle钩子来使用。
  ```js
  import React, { useRef, useImperativeHandle } from 'react';

  function FancyInput(props, ref) {
    const localInputRef = useRef();

    useImperativeHandle(ref, () => ({
      focus: () => {
        localInputRef.current.focus();
      },
      pageShow: ()=> {
  	//当外部监听到页面展示的时候可以调用页面内部的pageshow方法，完成页面内部状态的切换
      }
      // ...其他方法
    }));

    return <input ref={localInputRef} {...props} />;
  }

  // 在组件外部使用`forwardRef`进行包装
  const WrappedFancyInput = React.forwardRef(FancyInput);

  // 父组件中使用
  function ParentComponent() {
    const inputRef = useRef();

    return (
      <>
        <WrappedFancyInput ref={inputRef} />
        <button onClick={() => inputRef.current.focus()}>Focus input</button>
      </>
    );
  }

  ```
