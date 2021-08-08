### `React.PureComponent`

`React.PureComponent` 与 [`React.Component`](https://zh-hans.reactjs.org/docs/react-api.html#reactcomponent) 很相似。

两者的区别在于 

- `PureComponent` 中以浅比较` prop` 和 `state `的方式来实现了[`shouldComponentUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate)
- 而 `Component` 只要 `state `发生改变， 不论值是否与之前的相等，都会触发更新。

如果赋予 React 组件相同的 `props` 和 `state`，`render()` 函数会渲染相同的内容，那么在某些情况下使用 `React.PureComponent` 可提高性能。

### 什么是浅比较？

就是对比`当前状态（current）`和 `下一个状态（next）`下的` prop `和 `state`时，比较**基本数据类型**是否相同（如： 'a' === 'a'）, 而引用数据类型则是比较其引用地址是否相同，与值内容无关。

> 注意：在 `PureComponent` 中，当对传入的对象或者数组进行直接赋值时，因为并没有改变其引用地址，所以就不引起重新渲染。

#### 当比较数据是引用数据类型的情况

当`this.state.arr`是一个数组，且组件继承自`PureComponent`时。当点击按钮时，界面上没有变化，也没有输出`render`，证明没有渲染。但是我们可以从下面的注释中看到，每点击 一次按钮，我们想要修改的`arr`的值已经改变，而这个值将去修改`this.state.arr`，但是因为在`PureComponent`中浅比较这个数组的引用没有变化所以没有渲染，`this.state.arr`也没有更新，因为在`this.setState()`以后，值是在`render`的时候更新的， 这里涉及到`this.setState()`的知识

但是当这个组件是继承自`Component`的时候，初始化依旧是输出`constructor`和`render`，但是当点击按钮时，界面上出现了变化，即我们打印处理的`arr`的值输出，而且每点击一次按钮都会输出一次`render`，证明已经重新渲染，`this.state.arr`的值已经更新，所以 我们能在界面上看到这个变化

```jsx
import React, { PureComponent } from 'react';

class IndexPage extends PureComponent{
  constructor() {
    super();
    this.state = {
      arr:['1']
    };
  }
  changeState = () => {
    let { arr } = this.state;
    arr.push('2');
    console.log(arr);
    // ["1", "2"]
    // ["1", "2", "2"]
    // ["1", "2", "2", "2"] 
    // ....
    this.setState({
      arr
    })
  };
  render() {
    console.log('render');
    return (
      <div>
        <button onClick={this.changeState}>点击</button>
        <div>
          {this.state.arr.map((item) => {
            return item;
          })}
        </div>
      </div>
    );
  }
}

```

#### 如何处理这种情况

- 扩展运算符

下面的例子用`扩展运算符`产生新数组，使`this.state.arr`的引用发生了变化，所以每次点击按钮都会输出`render`，界面也会变化，不管该组件是继承自`Component`还是`PureComponent`的

```jsx
import React, { PureComponent } from 'react';

class IndexPage extends PureComponent{
  constructor() {
    super();
    this.state = {
      arr:['1']
    };
  }
  changeState = () => {
    let { arr } = this.state;
    this.setState({
      arr: [...arr, '2']
    })
  };
  render() {
    console.log('render');
    return (
      <div>
        <button onClick={this.changeState}>点击</button>
        <div>
          {this.state.arr.map((item) => {
            return item;
          })}
          </div>
      </div>
    );
  }
}
```

上面的情况同样适用于`对象`的情况。

- `immutable`

  `Immutable.js`就是一个`Immutable`库。这种情况下，属性的比较是非常容易的，因为已存在的对象就不会发生改变，取而代之的是重新创建新的对象。

### PureComponent不仅会影响本身，而且会影响子组件

我们让`IndexPage`组件里面包含一个子组件`Example`来展示`PureComponent`是如何影响子组件的

- 父组件继承`PureComponent`，子组件继承`Component`时：下面的结果初始化时输出为`constructor`，`IndexPage render`，`example render`，但是当我们点击按钮时，界面没有变化，因为这个`this.state.person`对象的引用没有改变，只是改变了它里面的属性值。所以尽管子组件是继承`Component`的也没有办法渲染，因为父组件是`PureComponent`，父组件没有渲染，子组件也不会渲染。

- 父组件继承`PureComponent`，子组件继承`PureComponent`时：因为渲染在父组件的时候就没有进行，相当于被拦截了，所以子组件是`PureComponent`还是`Component`根本不会影响结果，界面依旧没有变化

- 父组件继承`Component`，子组件继承`PureComponent`时：结果和我们预期的一样，即初始化是会输出`constructor`，`IndexPage render`，`example render`，但是点击的时候只会出现`IndexPage render`，因为父组件是`Component`，所以父组件会渲染，但是 当父组件把值传给子组件的时候，因为子组件是`PureComponent`，所以它会对`prop`进行浅比较，发现这个`person`对象的引用没有发生变化，所以不会重新渲染，而界面显示是由子组件显示的，所以界面也不会变化

- 父组件继承`Component`，子组件继承`Component`时：初始化是会输出`constructor`，`IndexPage render`，`example render`，当我们第一次点击按钮以后，界面发生变化，后面就不再改变，因为我们一直把它设置为`sxt2`，但是每点击一次 都会输出`IndexPage render`，`example render`，因为每次父组件和子组件都会渲染。

```jsx
//父组件
import React, { PureComponent, Component } from 'react';
import Example from "../components/Example";

class IndexPage extends PureComponent{
  constructor() {
    super();
    this.state = {
      person: {
        name: 'sxt'
      }
    };
    console.log('constructor');
  }
  changeState = () => {
    let { person } = this.state;
    person.name = 'sxt2';
    this.setState({
      person
    })
  };
  render() {
    console.log('IndexPage render');
    const { person } = this.state;
    return (
      <div>
        <button onClick={this.changeState}>点击</button>
        <Example person={person} />
      </div>
    );
  }
}
//子组件
import React, { Component } from 'react';

class Example extends Component {

  render() {
    console.log('example render');
    const { person } = this.props;
    return(
      <div>
        {person.name}
      </div>
    );
  }
}
```

### 总结

1. `PureComponent` 不仅会影响本身，而且会影响子组件，所以`PureComponent`最好作为展示组件。

2. 如果 `prop `和 `state` 每次都会变，那么使用 `Component` 的效率会更好，因为浅比较也是需要时间的。

3. 若有`shouldComponentUpdate`，则执行`shouldComponentUpdate`，若没有`shouldComponentUpdate`方法会判断是不是`PureComponent`，若是，进行浅比较。在`PureComponent`中使用`shouldComponentUpdate`会有警告。

   

### 注意

> `React.PureComponent` 中的 `shouldComponentUpdate()` 仅作对象的浅层比较。如果对象中包含复杂的数据结构，则有可能因为无法检查深层的差别，产生错误的比对结果。仅在你的 props 和 state 较为简单时，才使用 `React.PureComponent`，或者在深层数据结构发生变化时调用 [`forceUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#forceupdate) 来确保组件被正确地更新。你也可以考虑使用 [immutable 对象](https://facebook.github.io/immutable-js/)加速嵌套数据的比较。
>
> 此外，`React.PureComponent` 中的 `shouldComponentUpdate()` 将跳过所有子组件树的 prop 更新。因此，请确保所有子组件也都是“纯”的组件

