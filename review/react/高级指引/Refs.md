### 何时使用refs

当需要在典型数据流之外强制修改子组件。

- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。

避免使用 refs 来做任何可以通过声明式实现来完成的事情。

举个例子，避免在 `Dialog` 组件里暴露 `open()` 和 `close()` 方法，最好传递 `isOpen` 属性。

### 使用ref的三种方式

### 1.React.createRef（类组件使用）

对该节点的引用可以在 ref 的 `current` 属性中被访问。

`ref.current` 的值根据节点的类型而有所不同：

- 当 `ref` 属性用于 HTML 元素时，为底层 DOM 元素
- 当 `ref` 属性用于自定义 class 组件时，为组件的挂载实例
- **不能在函数组件上使用 `ref` 属性**，因为他们没有实例。但你可以在函数组件中使用 `ref`，只要它指向一个 DOM 元素或 class 组件

##### 为 DOM 元素添加 ref

```jsx
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个ref对象
    this.myRef = React.createRef();
  }

  componentDidMount() {
    // 使用ref访问DOM元素
    console.log(this.myRef.current);
  }

  render() {
    // 将ref对象赋值给DOM元素的ref属性
    return <div ref={this.myRef}>Hello, World!</div>;
  }
}
```

React 会在组件挂载时给 `current` 属性传入 DOM 元素，并在组件卸载时传入 `null` 值。`ref` 会在 `componentDidMount` 或 `componentDidUpdate` 生命周期钩子**触发前**更新。

##### 为 class 组件添加 ref

有时你可能需要调用子组件暴露的方法，可以给子组件添加ref来实现这一点

```jsx
class MyComponent extends React.Component {
  constructor(props) {
    super(props);

    this.childRef = React.createRef();
  }

  handleButtonClick = () => {
    // 调用子组件的方法
    this.childRef.current.someMethod();
  };

  render() {
    return (
      <div>
        <ChildComponent ref={this.childRef} />
        <button onClick={this.handleButtonClick}>Call Child Method</button>
      </div>
    );
  }
}

class ChildComponent extends React.Component {
  someMethod = () => {
    // ...
  };

  render() {
    // ...
  }
}
```

### 2. useRef（函数组件中使用）

```js
import React, { useRef } from 'react';

function MyFunctionalComponent() {
  const myRef = useRef(null);

  const handleClick = () => {
    if (myRef.current) {
      console.log(myRef.current); // 这里可以访问到DOM元素
    }
  };

  return (
    <div>
      <div ref={myRef}>Hello, World!</div>
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}

```

useRef使用起来和React.createRef有点像，让我们来比较一下

**useRef和React.createRef的异同**

- 相同点

1. 两者都用于获取DOM节点或组件实例的引用。
2. 返回的对象都有一个 `.current`属性，该属性指向绑定的DOM节点或类组件的实例。

- 不同点

1. 使用场景：

   * `React.createRef()`用于类组件。
   * `useRef`是一个Hook，专为函数组件设计。
2. 返回值稳定性：

   * 在类组件中，`React.createRef`每次调用都会创建一个新的ref对象，所以我们通常在构造函数中执行，以保证在组件的生命周期中持续使用同一个ref对象。
   * 在函数组件中，由于组件本质上是一个函数，它没有实例和构造函数，每次组件重新渲染都相当于重新执行了整个函数。Hooks如 `useRef`提供了一种机制，在多次重新渲染之间保持相同的ref对象。因此，`useRef`在创建时就是“持久化”的，即使在函数组件多次重新渲染时，它返回的ref对象也是同一个。
3. 性能影响：

   * `React.createRef()`在每次组件实例化时创建一个新的ref对象，不会在组件的更新中创建新的对象。
   * `useRef`避免了在每次渲染时重新创建ref对象的开销，因此它更适合在频繁重新渲染的函数组件中使用。

### 3. 回调 Refs（类组件和函数组件中都能使用）

回调Refs也可以在dom元素和类组件中使用。

传递给组件的ref属性值是一个函数。

将在组件挂载时，会调用 `ref` 回调函数并传入 DOM 元素，当卸载时调用它并传入 `null`。

React在 `componentDidMount` 或 `componentDidUpdate` 触发前，React 会保证 refs 一定是最新的。

这个函数中直接接受 React 组件实例或 HTML DOM 元素作为参数，不需要读取current属性值来获取。

```jsx
class MyComponent extends React.Component {
  componentDidMount() {
    // 使用ref访问DOM元素
    console.log(this.myRef);
  }

  render() {
    // 通过回调函数赋值ref
    return <div ref={ref => this.myRef = ref}>Hello, World!</div>;
  }
}

```

当使用回调refs并且回调函数是一个内联函数时。

- 在组件首次挂载时，内联函数的回调ref会被调用一次，并根据当前的DOM元素进行赋值。
- 在组件后续的每次更新中，如果你使用内联函数作为回调ref，内联函数的回调ref会被调用两次。第一次React会先用 `null`清除上一次的引用，第二次React再用新的DOM元素/组件实例调用回调函数更新ref。这是为了确保旧的DOM元素被正确地回收并且ref总是保持最新的值。

如果在每次渲染时都提供相同的回调函数实例给 `ref`属性，那么回调只会在挂载和卸载时执行一次。

1. 组件挂载时（第一次渲染到DOM），回调会被执行，接收到的参数是DOM元素。
2. 如果您在渲染期间始终提供相同的函数实例，那么在组件的更新过程中，回调不会被再次执行，因为React检测到ref的回调函数并没有改变。
3. 当组件被卸载时，回调会被执行，但这次接收到的参数是 `null`，该过程用于清理。

为了避免这个问题，可以使用类方法或箭头函数类属性，它们总是提供的是相同的回调函数实例。以下是两种方式的示例：

使用类方法：

```jsx
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.setMyRef = this.setMyRef.bind(this);
  }

  setMyRef(el) {
    this.myRef = el;
  }

  render() {
    return <div ref={this.setMyRef} />;
  }
}

```

使用箭头函数类属性：

```jsx
class MyComponent extends React.Component {
  setMyRef = (el) => {
    this.myRef = el;
  };

  render() {
    return <div ref={this.setMyRef} />;
  }
}

```

Ref转发

你可能希望在父组件中引用子节点的 DOM 节点。你可以用ref转发。

**其允许某些组件使用 `React.forwardRef` 接收 `ref`，并将其向下传递（换句话说，“转发”它）给子组件。**

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
