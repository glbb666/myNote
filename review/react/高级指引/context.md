### 何时使用 Context

Context 设计目的是为了共享那些对于一个组件树而言是“全局”的数据。适用于深层级，并且不同层级需要访问同样的数据时的情况。请谨慎使用，因为这会**使得组件的复用性变差**。

### 1. 创建一个 Context

你可以通过 `React.createContext()` 创建一个 Context 对象。通常你会将这个对象放在一个单独的文件中以便共享。

```js
// ThemeContext.js
import React from 'react';

const ThemeContext = React.createContext('light'); // 默认值为 'light'

export default ThemeContext;

```

#### 一些问题

为什么createContext要接受一个默认值呢

思考：

- 为了保证在组件树中如果找不到合适的 `Provider` 来匹配该 `Context` 时，组件仍能获取到一个从该 `Context` 中读取的值。
- 某些情况下，你可能需要在不包括整个 app 的结构，或者在一个不存在 `Provider` 的新上下文环境中测试或展示组件。默认值使得这个独立的组件在没有 `Provider` 存在时依然可以正确地展示和运行。

### 2. Context.Provider

将 `Context.Provider` 组件用作那些需要访问 `Context `数据的组件树的根。任何组件都可以访问到位于其组件树上方最近的 `Provider `中的 `value`。

```js
// App.js
import React from 'react';
import ThemeContext from './ThemeContext';
import Toolbar from './Toolbar';

function App() {
  return (
    // 使用一个 Provider 来传递当前的 value 值到下面的组件树中。
    // 无论多深，任何组件都能读取这个值。
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

```

### 3. 使用 Context的三种方法

#### 使用 Context.Consumer（类组件和函数组件都适用）：

**`ThemeContext.Consumer` 包含了一个函数作为其子元素，该函数接收当前的 context 值 `value` 作为参数**。这种模式允许你在渲染时读取最近的上游 `Provider` 提供的 context 值，如果没有上游 `Provider`，则会使用 `createContext` 时指定的默认值。

```js
// Button.js
import React from 'react';
import ThemeContext from './ThemeContext';

function Button() {
  return (
    // 使用一个 Consumer 组件来订阅最近的 theme context 的变化。
    <ThemeContext.Consumer>
      {value => /* 根据 context 值渲染某些东西 */}
    </ThemeContext.Consumer>
  );
}

```

#### 使用 useContext Hook（仅在函数组件中使用）

useContext直接就能取到value的值

```jsx
import React, { useContext } from 'react';
import ThemeContext from './ThemeContext';

function FunctionalButton() {
  const theme = useContext(ThemeContext); // 获取 context 的值
  return (
    <button style={{ background: theme === 'dark' ? 'black' : 'white' }}>
      I am also styled by theme context!
    </button>
  );
}

```

#### 使用Class.contentType（仅在类组件中使用）


在类组件中，我们可以通过静态属性 `contextType` 将 `Context` 赋给类，这样就可以在 `render` 方法和其他类方法中通过 `this.context` 访问 Context 的当前值。

此方法在编写类组件时较为方便，尤其是当你只需要从单一 context 源获取数据时。然而，它只能订阅一个 Context。d z

```jsx
// ThemedButton.js
import React, { Component } from 'react';
import ThemeContext from './ThemeContext';

class ThemedButton extends Component {
  static contextType = ThemeContext;

  render() {
    let theme = this.context; // 这里可以直接通过 this.context 访问 Context 的值
    return <button style={{ background: theme === 'dark' ? 'black' : 'white' }}>
        I am styled by theme context!
      </button>;
  }
}

export default ThemedButton;

```

### 注意

避免过度使用 Context。

Context 默认情况下会触发所有消费者的重渲染，即使是通过 `shouldComponentUpdate` 或 React.memo 优化的组件也不例外。

### 如何更新值

当你在 `Context` 中存储一个对象或数组时，如果每次更新都创建一个新的对象或数组（即使内容相同），那么使用 `Context.Consumer `或 `useContext` 的组件会认为 context 发生了变化（因为对象引用不同），并且这会导致所有消费该 Context 的组件重新渲染。

- badcase：以下代码会在每次渲染时创建一个新的 context 值，哪怕数据没变：

```jsx
<ThemeContext.Provider value={{ theme: 'dark' }}>
  {/* ... */}
</ThemeContext.Provider>

```

**好的实践：**

- 你可以使用 `useState` 的 setter 函数或者 `useReducer` 来保持相同的引用，除非数据实际发生了变化。举个例子：

```jsx
const [theme, setTheme] = useState({ theme: 'dark' }); // 初始化状态

// ...

<ThemeContext.Provider value={theme}>
  {/* 这个 Provider 提供的值 (`theme` 变量) 引用是稳定的，除非 `setTheme` 被调用 */}
</ThemeContext.Provider>

```

在这个例子中，只有当你调用 `setTheme` 来更新状态时，`theme` 变量的引用才会改变。

- 使用 `useReducer` 可以让你在指定的 ` reducer` 函数中集中处理所有的更新逻辑。这样，你的业务逻辑就与组件 `UI`解耦。`useReducer`可以和自定义 `hook`配合使用，复用状态逻辑。

  下面是一个完整的示例，包括如何在 Context 中使用 `useReducer` 并让组件能接收和 dispatch actions。

  首先，你需要创建一个 Context 并提供一个包含了 state 和 dispatch 函数的 value。这通常在一个封装了 `useReducer` 的上下文提供者（Provider）组件中完成。

  ```js
  // ThemeContext.js
  import React, { createContext, useReducer } from 'react';

  // 创建 Context 对象
  const ThemeContext = createContext();

  // 定义 reducer 函数
  function reducer(state, action) {
    switch (action.type) {
      case 'TOGGLE_THEME':
        return {
          ...state,
          theme: state.theme === 'dark' ? 'light' : 'dark',
        };
      default:
        return state;
    }
  }

  // 封装用于提供 Context 和 state 的 Provider 组件
  function ThemeProvider({ children }) {
    // 使用 useReducer，初始化 state
    const [state, dispatch] = useReducer(reducer, { theme: 'light' });

    // value 包含 state 和 dispatch
    const value = { state, dispatch };

    // 使用 ThemeContext.Provider 提供 Context
    return (
      <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
    );
  }

  export { ThemeProvider, ThemeContext };

  ```

  然后，你需要用 `ThemeProvider` 包裹你的应用或组件树的那部分，以便下层组件可以访问到 Context 中的 state 和 dispatch：

  ```js
  // App.js
  import React from 'react';
  import { ThemeProvider } from './ThemeContext';
  import MyComponent from './MyComponent';

  function App() {
    return (
      <ThemeProvider>
        <MyComponent />
      </ThemeProvider>
    );
  }

  export default App;

  ```

  现在，在你的 `MyComponent` 里，你可以使用 `useContext` 来订阅 `ThemeContext` 从而获取 state 和 dispatch：

  ```js
  // MyComponent.js
  import React, { useContext } from 'react';
  import { ThemeContext } from './ThemeContext';

  function MyComponent() {
    // 使用 useContext 获取 state 和 dispatch
    const { state, dispatch } = useContext(ThemeContext);

    return (
      // 当按钮被点击时，dispatch 一条 TOGGLE_THEME 的 action 来改变主题
      <button onClick={() => dispatch({ type: 'TOGGLE_THEME' })}>
        Toggle Theme
      </button>
    );
  }

  export default MyComponent;

  ```

  在类组件中，你通常会使用 `this.state` 和 `this.setState` 方法来处理状态。

### 一些问题

Q1：怎么定义消费者？

“消费者（consumer）”是指那些读取或使用了 Context 值的组件。具体来说，一个组件如果使用了 `Context.Consumer` 组件、`useContext` Hook 或者在类组件中通过 `this.context` 来获取 Context 的值，那么这个组件被认为是消费者。仅仅被 `Provider` 包裹并不会自动使一个组件成为消费者，它必须直接使用了 Context 中的值才算。

Q2：Context是有代价的，在什么场景下必须用Context

1. **主题切换** ：
   主题切换是用户界面状态的一部分，要求在应用内实时变更并响应用户交互，`localStorage` 可以用来持久化用户的主题偏好，但仍需 Context 或其他状态管理器来管理实时状态。
2. **表单数据管理** ：
   表单通常是实时交互的，并且经常依赖组件的即时状态。保存表单的即时状态到 `localStorage` 是不必要的，而且可能会引发问题，比如与用户当前操作不同步。
3. **应用状态管理** ：
   比如音乐播放器的实时状态，需要随时根据用户交互更新，不适合使用 `localStorage` 来管理这种类型的实时状态。
4. **交互状态记录** ：
   记录用户操作的历史或交互状态通常是为了即时反馈，并实现撤销/重做等功能。`localStorage` 可能不足以处理这类快速和频繁的更新。
