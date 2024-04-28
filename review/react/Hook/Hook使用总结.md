### Hook是什么

React的Hook是React 16.8中引入的新功能，它允许你在不编写类组件的情况下使用React特性。常用的React Hook包括useState, useEffect, useContext等。

### 为什么使用Hook

1. 简化代码：Hook允许你在不使用类的情况下使用React的特性。
2. 可复用性：自定义Hook可以被多个组件复用，这减少了代码冗余和复杂性。
3. 清晰的逻辑：使用Hook，你可以按逻辑收集相关的代码，而不是按照生命周期方法分离它们。
4. 减少嵌套：Hook减少了高阶组件和render props模式的需要，这两者通常会导致组件树的深层嵌套。
5. 拥抱函数式编程：Hook鼓励你使用更多的函数式编程概念，比如纯函数和不可变性，这可以提高应用的稳定性和性能。

### **React 中提供的 hooks：**

#### 1. State Hooks

* useState
* useReducer：setState，useState 是该方法的封装

  ```js
  const [state, dispatch] = useReducer(reducer, initialState);
  ```

#### 2.Effect Hooks

* useEffect: 类似 componentDidMount/Update, componentWillUnmount，当效果为 componentDidMount/Update 时，总是在整个更新周期的最后（页面渲染完成后）才执行
* useLayoutEffect: 用法与 useEffect 相同，区别在于该方法的回调会在数据更新完成后，页面渲染之前进行，该方法会阻碍页面的渲染

#### 3. context Hooks

* useContext: context，需配合 createContext 使用

#### 4. ref Hooks（高级指引）

* useRef: ref
* useImperativeHandle: 给 ref 分配特定的属性

#### 5. 性能优化 Hook（Hook里有提到）

* useMemo: 可以对 setState 的优化
* useCallback: useMemo 的变形，对函数进行优化

### 注意

* 不能将 hooks 放在循环、条件语句或者嵌套方法内。hooks的底层实现机制是双向链表，react 是根据 hooks 出现顺序来记录对应状态的。
