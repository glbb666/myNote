### 简介

JSX用于描述组件的UI，JSX将html语法直接加入 JavaScript 中，再通过翻译器转换到纯JavaScript后由浏览器执行，让代码更加直观并且易于维护。它的本质并不是模版语言，而是 `React.createElement`的语法糖，方便你动态创建组件。

### jsx表示对象的两种方式

Babel 会把 JSX 转译成一个名为 `React.createElement()` 函数调用。

以下两种示例代码完全等效：

```jsx
//方式1，这个是jsx
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
//方式2 React.createElement('标签名称,标签属性,子元素')，这个是虚拟DOM
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
//实际上它创建了一个这样的对象：
// 注意：这是简化过的结构
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```

### jsx的{}中不支持⚠️

- 不支持多行JavaScript语句，{}中只支持表达式

```jsx
const el = <Mycomponent foo = {const value = 1+2;return value};
```

### jsx中的一些改进

#### 标签属性

- DOM标签属性：因为jsx的风格更接近于JavaScript，所以onblur，onclick，onfocus的写法均变为驼峰形式的**onBlur，onClick，onFocus**，同时因为class是react的关键词，所以**class变为className**
- 自定义标签属性：支持自定义标签属性，有以下两种方式

  - “”，字符串字面量

  ```jsx
  const element = <div tabIndex="0"></div>;
  ```

  - {}，JavaScript表达式

  ```jsx
  const element = <img src={user.avatarUrl}></img>;
  ```

  ⚠️注意：“”和 {}不能同时使用

#### 注释

jsx中的注释需要用{}将/**/包起来

```jsx
{/*这是一个注释*/}
```
