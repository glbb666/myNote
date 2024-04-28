React 应用中有多种方式可以实现组件之间的通信，这些方法适用于不同的场景和需求。下面是一些常见的组件通信方式：

### **Props 传递** ：(父->子)

* 父组件向子组件传递数据和回调函数。
* 子组件通过 props 接收数据和函数，并且可以通过调用这些传入的函数与父组件通信。

### **回调函数** ：（子->父）

* 父组件可以将回调函数作为 props 传递给子组件。
* 子组件在某些事件发生时执行这些回调函数来传递数据或通知父组件。

### **组件提升（Lifting State Up）** ：（组件间）

* 共享的状态被提升至最近的共同祖先组件中管理。
* 在这个祖先组件中处理状态变更逻辑，然后通过 props 下发状态或回调。

### **Context API** ：（这个有专门写一篇）

* 提供了一种在组件树中共享值如主题或用户认证信息的方式，而不必显式通过组件链逐层传递 props。

### **组合组件：**

父组件可以在子组件中使用特定的props来控制它的行为：

1. 数据Props：父组件可以传递数据给子组件，这些数据可以用于决定子组件的渲染内容或样式。
2. 事件处理Props：父组件可以将回调函数作为props传递给子组件。子组件在特定事件发生时（如用户点击按钮）可以调用这些回调函数，从而触发父组件中的逻辑。
3. 渲染回调Props或子组件插槽：有时候父组件需要决定子组件的部分内容或结构。父组件可以通过渲染回调（将React元素作为函数参数）或者“插槽”（预留位置让父组件填充）的方式将渲染的控制权交给子组件。

详细说一下第三点：

#### 渲染回调

渲染回调是一种在React组件间共享代码的技术。子组件接收一个函数作为props，然后这个函数返回React元素。通常这个函数prop被命名为 `render`。父组件在实现的时候，将决定子组件部分内容的代码作为函数参数传递给子组件。子组件内部在合适的位置调用这个函数，并使用返回的React元素来渲染。

```jsx
class ParentComponent extends React.Component {
  render() {
    return (
      <ChildComponent render={(someData) => (
        <CustomComponent data={someData} />
      )} />
    );
  }
}

class ChildComponent extends React.Component {
  render() {
    // 从props中解构出render函数
    const { render } = this.props;
    // 一些可供渲染使用的数据
    const someData = "Some data to be used by the rendering function";
  
    // 在合适的位置调用render函数，并传入数据
    return (
      <div>
        {render(someData)}
      </div>
    );
  }
}

```

#### 插槽（Slots）:

子组件作为容器，在定义结构的时候预留位置，由父组件决定填充什么内容。

我们可以使用props.children 或者 通过特定的prop 来实现插槽。请看示例。

特定的props一般直接是一个特定的内容，不像渲染回调，是一个render函数。

```jsx
class ParentComponent extends React.Component {
  render() {
    return (
      <ChildComponent
  	header={<h1>This is a header</h1>}
      >
        <h1>This is a header slot</h1>
        <p>This is a paragraph slot</p>
      </ChildComponent>
    );
  }
}

class ChildComponent extends React.Component {
  render() {
    return (
      <div>
        {/* 这种属于传统的插槽 */}
        {this.props.children}
	{/* 这种属于插槽的变体 */}
        <div className="header">{this.props.header}</div>
      </div>
    );
  }
}

```

渲染函数和插槽的用法其实有一些重合，为了便于区分，我们可以这样理解。

* **渲染函数** （Render Function）通常是一个以 `render`命名的prop，它期望得到一个函数，这个函数返回React元素。这个函数通常会接受一些参数（比如组件的状态或props），然后用这些参数来决定最终渲染的内容。

```jsx
<SomeComponent render={(data) => <div>{data}</div>} />
```

* **插槽** （Slots）则是利用 `children` prop或其他自定义的prop，传递静态或动态的React元素，这些元素将在子组件的特定部分被直接渲染。

```jsx
<SomeComponent>
  <div>这是通过children prop传递的内容</div>
</SomeComponent>
```

或者：

```jsx
<SomeComponent content={<div>这是通过自定义prop传递的内容</div>} />
```

将**渲染回调显式命名为 `render`**和使用 **`children`代表实际的子元素的插槽**，能够帮助区分组件的不同用途，并避免混淆其意图。这是建议的最佳实践。

### **Refs** ：(这个有专门写一篇)

* 使用 React 提供的 `React.createRef()` 或 `useRef()` Hook 来获取子组件的实例或 DOM 元素，并对其进行操作。

### **事件发布/订阅模型** ：（高级指引/EventBus）

* 使用自定义事件系统或者第三方库（如 mitt、EventEmitter3 等）进行非关联组件间的通信。

### 广播
广播和EventBus的机制类似，不过EventBus在每个页面都会创建一个新的实例。对于mpa架构的项目而言，广播可以进行跨页面共享状态或保持事件监听。

### **全局状态管理库** ：（redux目录）

* Redux提供了跨组件共享和管理状态的能力。
* 通过定义全局的 actions 和 reducers（或其他实体），应用内的任意组件都可以访问和更新状态。
