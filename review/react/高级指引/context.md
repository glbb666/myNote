### 何时使用 Context

Context 设计目的是为了共享那些对于一个组件树而言是“全局”的数据。适用于深层级，并且不同层级需要访问同样的数据时的情况。请谨慎使用，因为这会**使得组件的复用性变差**。

### api

```jsx
//创建一个context对象
const MyContext = React.createContext(defaultValue);
//context对象都会有一个Provider的组件，接受一个value作为context的数据源，被Provider包裹的子组件都可以使用value的值
//消费组件：被Provider包裹，并且使用value值的组件
<MyContext.Provider value={/*某个值*/}></MyContext.Provider>
//消费组件消费value的两种方式
//1.Class.contentType
//将创建出的context对象作为消费组件的属性值，在类组件的任意生命周期中都可以通过访问this.context来访问到Provider的value值
myClass.contentType = MyContext;
//2.Context.Consumer
//在消费组件外面包裹一个Context.Consumer组件，内部接受函数作为子元素，函数的参数value就是context对象，返回消费组件。
<MyContext.Consumer>
{value=>/*基于context值进行渲染*/}
</MyContext.Consumer>
```

简单的使用🌰

```jsx
const ThemeContext = React.createContext('light');
class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}
// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar(props) {
  return (
      <ThemedButton />
   );
}
class ThemedButton extends React.Component {
  render() {
    return <Button theme={this.context} />;
  }
}
ThemedButton.contentType = MyContext;
```

⚠️注意：

- context的默认值只在外层<font color='red'>无Provider包裹</font>时被使用，如果有Provider但是没有传值，依然会从Provider上读取值
- Provider支持<font color='red'>多层嵌套</font>，但是最好把状态都放在一个Provider内，不要嵌套层级太深
- 消费组件的更新只取决于Provider值的更新，不受shouldComponentUpdate限制

