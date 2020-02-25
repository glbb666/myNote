## 组件

组件类似于JavaScript函数，<font color='red'>必须以大写字母开头</font>。

### 两种组件

- 函数组件，接收**唯一带有数据**的 “props”（代表属性）对象与并返回一个 React 元素。

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

- class组件

将函数组件转成 class 组件：

1. 创建一个同名的 [ES6 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes)，并且继承于 `React.Component`。
2. 将函数体移动到 `render()` 方法之中。
3. 在 `render()` 方法中使用 `this.props` 替换 `props`。

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

⚠️注意

- 不管是哪种方式返回的组件，组件有且仅有一个根元素。

- 且组件返回的是**react元素**。

  举个🌰 ，加深一下对组件和元素区别的理解。

  ```jsx
  <Parent>
  	<Children>我是子组件</Children>
  </Parent>
  ```

  我们可以这样写

  ```jsx
  import Children from 'Children'
  
  class Parent extends React.Component{
  	render(){
  		const { tip } = this.props;
  		return <Children tip={tip}/>
  	}
  }
  ```

  另一种，因为组件的子元素都挂载在`this.props`对象上，可以通过`this.props.children`获取到组件的子元素，那么可以不可以把它写成jsx的形式呢

  ```jsx
  //wrong
  class Parent extends React.Component{
  	render(){
  		const { tip,children } = this.props;
  		return <children tip={tip}/>
  	}
  }
  ```

   this.props.children获取到的是react元素，不是react组件。

  我们可以使用`React.cloneElement`来使用和处理react元素

  ```jsx
  class Parent extends React.Component{
  	render(){
  		const { tip,children } = this.props;
  		return React.cloneElement(children,{
  			tip,
  		});
  	}
  }
  ```

### 提取组件的时机

根据经验来看，如果 UI 中有一部分被多次使用（`Button`，`Panel`，`Avatar`），或者组件本身就足够复杂（`App`，`FeedStory`，`Comment`），那么它就是一个可复用组件的候选项。 

## props

组件的参数，由组件外部传入，唯一，只读。

### 属性校验

可以使用`PropTypes`包对外部传入的属性值进行校验，有两种方式

- 方式一：在组件内部用`static`声明一个静态属性`propTypes`

```jsx
import PropTypes from 'prop-types';

class CommentItem extends React.Component{
	static propTypes = {
		userImg:PropTypes.string.isRequired,
    //isRequired表示这个值是必须的，如果没有传入或不符合要求会warning
		likeCount:PropTypes.number
	};
}
```

- 方式二：在组件外部在类的上名定义一个`propTypes`的属性值

```jsx
import PropTypes from 'prop-types';

class CommentItem extends React.Component{
}
CommentItem.propTypes = {
		userImg:PropTypes.string.isRequired,
		likeCount:PropTypes.number
};
```

### 默认属性

当`react`收到的属性值为`undefined`，或父组件没有传入`props`，组件中的默认属性值就会读取设置的`defaultProps`的值，`defaultProps`的值是组件在初始化的生命周期中被初始化的，它也有两种写法

- 方式一：

```jsx
CommentItem.defaultProps = {
	userImg:'',
	nickName:'',
	likeCount:0
}
```

- 方式二：

```jsx
static defaultProps = {
	userImg:'',
	nickName:'',
	likeCount:0
}
```

## className

官方推荐的简单方式

- 单个className，字符串形式

- 多个className

  - 模版字符串

  ```jsx
  <img className = {`heart ${className}`}
  ```

  - 第三方的classnames，动态加载className

  ```jsx
  import classNames from 'classnames';
  //使用规则如下
  classNames('foo','bar');//=>'foo bar'
  classNames('foo',{'bar':true});//=>'foo bar'
  classNames('foo-bar':true);//=>'foo-bar'
  classNames('foo-bar':false);//=>''
  classNames({'foo':true},{'bar':true});//=>'foo bar'
  classNames('foo':true,'bar':true});//=>'foo bar'
  
  const btnClass = classNames({
  	'btn':true,
  	'btn-pressed':this.state.isPressed,
  	'btn-over':!this.state.isPressed && this.state.isHovered
  });
  ```

### 文件作用域

- 全局

```jsx
<link rel='stylesheet' type='text/css' href='style.css'/>
```

- 组件作用域

```jsx
import './style.css'
```

## css-in-JS的行内样式

特定场景使用，比如需要动态计算样式值的时候。

属性名用驼峰命名，属性值可以用字符串或数字，可以参略px单位

```jsx
const styles = {
	container:{
		background:'#f6f6f8 url(/images/bg.png) repeat-x';
		borderRadius:2
	},
	depressed:{
		backgroundColor:'#4e69a2',
		borderColor:'#c6c7ca',
		color:'#5890ff'
	},
}

<button style={styles.container}>
	<span style={styles.depressed}>确定</span>
</button>
```

